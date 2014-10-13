#log4j输入日志到syslog

##log4j配置
>log4j.rootLogger=syslog
log4j.appender.syslog=org.apache.log4j.net.SyslogAppender
log4j.appender.syslog.SyslogHost=221.123.169.78
log4j.appender.syslog.Facility=skullcandy
log4j.appender.syslog.Threshold=DEBUG
log4j.appender.syslog.layout=org.apache.log4j.PatternLayout
log4j.appender.syslog.layout.ConversionPattern=[%-d{yyyy-MM-dd HH:mm:ss}:skullcandy_statistics][%t]%l%n[%p]%m



