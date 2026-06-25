[root@omnidnc conf.d]# [Thu Jun 25 19:23:56.341527 2026] [:error] [pid 146644:tid 140114088871680] [client 211.104.36.161:8773] [client 211.104.36.161] ModSecurity: Warning. Operator GE matched 5 at TX:inbound_anomaly_score. [file "/etc/httpd/modsecurity.d/activated_rules/RESPONSE-980-CORRELATION.conf"] [line "92"] [id "980130"] [msg "Inbound Anomaly Score Exceeded (Total Inbound Score: 5 - SQLI=5,XSS=0,RFI=0,LFI=0,RCE=0,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 5, 0, 0, 0"] [ver "OWASP_CRS/3.3.4"] [tag "event-correlation"] [hostname "vitamall.omnidnc.co.kr"] [uri "/phpMyAdmin/import.php"] [unique_id "aj0BvGoRQqAZ819un3WVmAAAAFg"], referer: https://vitamall.omnidnc.co.kr/phpMyAdmin/index.php?token=a168795caef725966088528638db2554&set_fontsize=103%25^C

modsecurity웹 방화벽에서 접속아이피가 차단되서 phpmyadmin 쿼리조회가 안되던 오류
sudo vi /etc/httpd/conf.d/modsecurity_localrules.conf -> 예외규칙설정
# 특정 관리자 IP에 대해 ModSecurity 검사 비활성화
SecRule REMOTE_ADDR "@ipMatch 211.104.36.161" \
    "id:1000001,\
    phase:1,\
    pass,\
    nolog,\
    ctl:ruleEngine=Off"

httpd -t로 문법오류 검사후
아파치 재시작