# AirFlow

https://airflow.apache.org

关于AirFlow和Beam的对比：

- https://www.astronomer.io/blog/airflow-vs-apache-beam/
- https://stackoverflow.com/questions/50249759/apache-airflow-or-apache-beam-for-data-processing-and-job-scheduling

如果以Airflow为主体搭建：

1. 不动核心模块，在此基础上通过插件（AirFlow Plugin）拓展功能。
2. 二次开发Airflow的核心模块，如果确实无法满足我们需求。

主要计划拓展的特性是：

1. 腾讯云等国内云厂商，主要是云原生相关。
2. 企业微信等国内的企业账号，比如WebHook消息通知。
