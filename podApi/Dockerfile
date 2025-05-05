FROM alpine
ADD podApi /podApi
ADD filebeat.yml /filebeat.yml
#filebeate二进制文件需要下载打进镜像里
#Add filebeat /filebeat
ENTRYPOINT [ "/podApi" ]