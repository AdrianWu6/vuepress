---
title: 基于MySQL数据库的序号自增方式
date: 2026-07-20
categories:
  - DataBase
tags:
  - MySql
---
## 问题背景
项目上一个外部接口需要序号自增的六位流水号作为唯一流水号，目前没有redis等外部中间件可以借用。Java版本为jdk1.8，数据库框架为Spring JDBCTemplate，数据库为Mysql 8.0.11，我们的系统有三台负载。
## 实现方式
单独建立一张“每日流水号计数表”。

每天只保存一条记录：

```
seq_date    current_value
2026-07-15  125
```

其中：

- `seq_date`：日期，也是主键；
- `current_value`：当天已经生成到的序号；
- 第二天日期变化后，自动插入新记录，从 `1` 开始；
- 三台服务器统一操作同一个 MySQL 主库；
- MySQL 负责并发控制和原子递增。
## 建表语句
```sql
CREATE TABLE biz_daily_sequence (
    seq_date DATE NOT NULL COMMENT '流水日期',
    current_value INT UNSIGNED NOT NULL COMMENT '当天当前流水号',
    updated_at DATETIME(3) NOT NULL COMMENT '最后更新时间',
    PRIMARY KEY (seq_date)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COMMENT = '每日流水号计数表';
```
## 代码实现
```java
import org.springframework.jdbc.core.ConnectionCallback;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

@Service
public class DailySequenceService {

    private static final long MAX_SEQUENCE = 999999L;

    private static final String GENERATE_SQL =
            "INSERT INTO biz_daily_sequence (" +
            "    seq_date, " +
            "    current_value, " +
            "    updated_at " +
            ") VALUES (" +
            "    CURRENT_DATE(), " +
            "    LAST_INSERT_ID(1), " +
            "    NOW(3) " +
            ") " +
            "ON DUPLICATE KEY UPDATE " +
            "    current_value = CASE " +
            "        WHEN current_value < 999999 " +
            "            THEN LAST_INSERT_ID(current_value + 1) " +
            "        ELSE current_value + LAST_INSERT_ID(0) * 0 " +
            "    END, " +
            "    updated_at = NOW(3)";

    private static final String QUERY_RESULT_SQL =
            "SELECT " +
            "    DATE_FORMAT(CURRENT_DATE(), '%Y%m%d') AS business_date, " +
            "    LAST_INSERT_ID() AS sequence_value";

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public DailySequenceService(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    /**
     * 生成六位流水号，例如：000001。
     */
    public String nextSixDigitSequence() {
        SequenceResult result = generateSequence();

        return String.format("%06d", result.getSequenceValue());
    }

    /**
     * 生成日期加六位流水号，例如：20260715000001。
     */
    public String nextBusinessNo() {
        Long result = generateSequence();

        return String.format("%06d", result);
    }

    private Long generateSequence() {
        Long result = jdbcTemplate.execute(
                new ConnectionCallback<Long>() {
                    @Override
                    public Long doInConnection(Connection connection)
                            throws SQLException {

                        /*
                         * 两条SQL必须使用同一个物理数据库连接。
                         *
                         * LAST_INSERT_ID(expr)保存的是当前连接级别的值。
                         */
                        try (Statement statement = connection.createStatement()) {

                            statement.executeUpdate(GENERATE_SQL);

                            try (ResultSet resultSet =
                                         statement.executeQuery(QUERY_RESULT_SQL)) {

                                if (!resultSet.next()) {
                                    throw new DailySequenceException(
                                            "未获取到流水号生成结果"
                                    );
                                }

                                long sequenceValue =
                                        resultSet.getLong("sequence_value");

                                if (sequenceValue <= 0) {
                                    throw new DailySequenceException(
                                            "日期 " + businessDate
                                                    + " 的六位流水号已达到上限 "
                                                    + MAX_SEQUENCE
                                    );
                                }

                                return sequenceValue;
                            }
                        }
                    }
                }
        );

        if (result == null) {
            throw new DailySequenceException("流水号生成结果为空");
        }

        return result;
    }
}
```
## 会不会造成严重锁表
- 表使用 InnoDB；
- 日期是主键；
- 更新能够通过主键直接定位；
- 不需要扫描整张表。

因此不是严重的表锁，而是当天计数行的短暂竞争。