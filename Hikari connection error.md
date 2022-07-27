# Connection is not available, request timed out after 30000ms



현대이지웰 프로젝트에서 adSystem에서 노출과 클릭 서비스 도중 300원이 더빠져서 확인하다보니 해당 에러 로그 확인



원인 : Querydsl에서 transform을 사용하면서 DB connection leak이 걸림. querydsl에서 transform 사용 시 트랜잭션 내보에서 실행되지 않는다면 해당 db connection은 해제 되지 않음 --> 구글 검색



HicariCP의 역할은 Database connection pool을 관리, 



1. HikariCP는 서버와 데이터베이스의 연결을 위해 미리 정해놓은 만큼 connection을 만들어서 pool에 담아 놓음
2. 요청이 들어오면 Thread가 connection을 요청하고, HikariCP내에 있는 coonection 제공
3. Thread는 connection을 사용하고 나서 pool로 다시 반환

hikar-cp default value



https://cljdoc.org/d/hikari-cp/hikari-cp/2.14.0/doc/readme



| Option                      | Required | Default value          | Description                                                  |
| --------------------------- | -------- | ---------------------- | ------------------------------------------------------------ |
| `:auto-commit`              | No       | `true`                 | This property controls the default auto-commit behavior of connections returned from the pool. It is a boolean value. |
| `:read-only`                | No       | `false`                | This property controls whether Connections obtained from the pool are in read-only mode by default. |
| `:connection-timeout`       | No       | `30000`                | This property controls the maximum number of milliseconds that a client will wait for a connection from the pool. If this time is exceeded without a connection becoming available, a SQLException will be thrown. 1000ms is the minimum value. |
| `:validation-timeout`       | No       | `5000`                 | This property controls the maximum amount of time that a connection will be tested for aliveness. This value must be less than the `:connection-timeout`. The lowest accepted validation timeout is 1000ms (1 second). |
| `:idle-timeout`             | No       | `600000`               | This property controls the maximum amount of time (in milliseconds) that a connection is allowed to sit idle in the pool. |
| `:max-lifetime`             | No       | `1800000`              | This property controls the maximum lifetime of a connection in the pool. A value of 0 indicates no maximum lifetime (infinite lifetime). |
| `:minimum-idle`             | No       | `10`                   | This property controls the minimum number of idle connections that HikariCP tries to maintain in the pool. |
| `:maximum-pool-size`        | No       | `10`                   | This property controls the maximum size that the pool is allowed to reach, including both idle and in-use connections. Basically this value will determine the maximum number of actual connections to the database backend. |
| `:pool-name`                | No       | Auto-generated         | This property represents a user-defined name for the connection pool and appears mainly in logging and JMX management consoles to identify pools and pool configurations. |
| `:jdbc-url`                 | **Yes¹** | None                   | This property sets the JDBC connection URL. Please note the `h2` adapter expects `:url` instead of `:jdbc-url`. |
| `:driver-class-name`        | No       | None                   | This property sets the JDBC driver class.                    |
| `:adapter`                  | **Yes¹** | None                   | This property sets the database adapter. Please check [Adapters and corresponding datasource class names](https://cljdoc.org/d/hikari-cp/hikari-cp/2.14.0/doc/readme#adapters-and-corresponding-datasource-class-names) for the full list of supported adapters and their datasource class names. The value should be provided as a string. |
| `:username`                 | No       | None                   | This property sets the default authentication username used when obtaining Connections from the underlying driver. |
| `:password`                 | No       | None                   | This property sets the default authentication password used when obtaining Connections from the underlying driver. |
| `:database-name`            | No       | None                   | This property sets the database name.                        |
| `:server-name`              | No       | Depends on the adapter | This property sets the hostname client connects to.          |
| `:port-number`              | No       | Depends on the adapter | This property sets the port clients connects on.             |
| `:connection-test-query`    | No       | None                   | If your driver supports JDBC4 we strongly recommend not setting this property. This is for "legacy" databases that do not support the JDBC4 `Connection.isValid()` API. This is the query that will be executed just before a connection is given to you from the pool to validate that the connection to the database is still alive. |
| `:leak-detection-threshold` | No       | 0                      | This property controls the amount of time that a connection can be out of the pool before a message is logged indicating a possible connection leak. A value of 0 means leak detection is disabled, minimum accepted value is 2000 (ms). ( *ps: it's rarely needed option, use only for debugging* ) |
| `:register-mbeans`          | No       | false                  | This property register mbeans which can be used in jmx to monitor hikari-cp. |
| `:connection-init-sql`      | No       | None                   | This property sets a SQL statement that will be executed after every new connection creation before adding it to the pool. |
| `:datasource`               | No       | None                   | This property allows you to directly set the instance of the DataSource to be wrapped by the pool. |
| `:datasource-class-name`    | No       | None                   | This is the name of the DataSource class provided by the JDBC driver. |
| `:metric-registry`          | No       | None                   | This is an instance of a Dropwizard metrics MetricsRegistry. |
| `:health-check-registry`    | No       | None                   | This is an instance of a Dropwizard metrics HealthCheckRegistry. |
| `:metrics-tracker-factory`  | No       | None                   | This is an implementation of a `MetricsTrackerFactory` (`CodahaleMetricsTrackerFactory`, `MicrometerMetricsTrackerFactory`, `PrometheusMetricsTrackerFactory`). |
| `:transaction-isolation`    | No       | None                   | Constant name from the [Connection](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html) interface such as `TRANSACTION_READ_COMMITTED`, `TRANSACTION_REPEATABLE_READ`, etc. |





connection-timeout은 pool에서 connection을 얻어오기까지 기다리는 최대 시간으로 허용 가능한 시간을 초과하면 SQLException이 발생합니다.

(default 값은 30000ms으로 30s가 됩니다.)



maximum-pool-size 옵션은 pool에 유지시킬 수 있는 최대 connection을 지정하는 옵션입니다.

(default 값은 10입니다.)



pool에 있는 connection을 요청했지만 사용중인 maximum-pool-size가 10개 모두 다 사용중이라 30초동안 요청해도 connection을 얻을 수 없어 sqlException 반환



해결방안 : 1. transform을 트랜잭션 내부에서 실행, 2. transform을 사용하지 않는 것