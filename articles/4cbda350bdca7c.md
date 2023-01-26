---
title: "ドメインの有効期限・ステータスを監視するシェルスクリプト"
emoji: "🚀"
type: "tech"
topics:
  - "shell"
  - "domain"
  - "cron"
published: true
published_at: "2021-12-05 01:38"
---

# 概要

監視結果をChatWorkに通知します
http://iarai.seesaa.net/article/135303163.html

## やったこと

```sh:domain_watch.sh
#!/bin/bash
PROGPATH=`echo $0 | /bin/sed -e 's,[\\/][^\\/][^\\/]*$,,'`
#. $PROGPATH/utils.sh
# Default values (days):
critical=30
warning=60
whois="/usr/bin/whois"
host=""
MESSAGE=""
# 監視対象のドメインを追加してください
domains=(
google.com
 yahoo.jp
)

MAXCOUNT=${#domains[@]}

for v in "${domains[@]}"
do
    # 連続実行するとwhoisがエラーになるので対応
    sleep 15;
    
    # 監視対象実行件数カウント
    count=$((count+1))
    
    # Parse arguments
    args=`getopt -o hd:w:c:W:H: --long help,domain:,warning:,critical:,whois:,host: -u -n $0 -- "$@"` 
    [ $? != 0 ] && echo "$0: Could not parse arguments" && echo "Usage: $0 -h | -d <domain> [-W <comman>] [-c <critical>] [-w <warning>]" && exit
    set -- $args
        while true ; do
            case "$1" in
                    -h|--help)      usage;exit;;
                    -d|--domain)    domain=$2;shift 2;;
                    -w|--warning)   warning=$2;shift 2;;
                    -c|--critical)  critical=$2;shift 2;;
                    -W|--whois)     whois=$2;shift 2;;
                    -H|--host)      host="-h $2";shift 2;;
                    --)             shift; break;;
                    *)              echo "Internal error!" ; exit 1 ;;
            esac
        done
    [ -z $v ] && echo "UNKNOWN - There is no domain name to check" && exit $STATE_UNKNOWN
    
    # Looking for whois binary
    if [ ! -x $whois ]; then
        echo "UNKNOWN - Unable to find whois binary in your path. Is it installed? Please specify path."
        exit $STATE_UNKNOWN
    fi
    
    # ドメイン名を取得 .xx.xxも取得
    TLDTYPE=`echo ${v#*.}| tr '[A-Z]' '[a-z]'`
    echo "host:$v"
    
    # 期限日を取得
    if [ "${TLDTYPE}" == "in" -o "${TLDTYPE}" == "info" -o "${TLDTYPE}" == "org" ]; then
        expiration=`$whois $host $v | awk '/Expiration Date:/ { print $2 }' |cut -d':' -f2`
    elif [ "${TLDTYPE}" == "biz" ]; then
        expiration=`$whois $host $v | awk '/Domain Expiration Date:/ { print $6"-"$5"-"$9 }'`
    elif [ "${TLDTYPE}" == "sc" ]; then
        expiration=`$whois -h whois2.afilias-grs.net $host $v | awk '/Expiration Date:/ { print $2 }' | awk -F : '{ print $2 }'`
    elif [ "${TLDTYPE}" == "jp" -o "${TLDTYPE}" == "jp/e" -o "${TLDTYPE}" == "co.jp" -o "${TLDTYPE}" == "or.jp" ]; then
        expiration=`$whois $host $v | awk '/Expires/ { print $NF }'`
        if [ -z $expiration ]; then
            expiration=`$whois $host $v | awk '/State/ { print $NF }' | tr -d \(\)`
        fi
    else
            expiration=`$whois $host $v | awk '/Expiration/ { print $NF }'`
    fi
    echo $expiration
    expseconds=`date +%s --date="$expiration"`
    nowseconds=`date +%s`
    ((diffseconds=expseconds-nowseconds))
    expdays=$((diffseconds/86400))
    
    #チャットワーク通知タイトル
    TITLE="ドメイン監視"
    
    #チャットワーク通知文言
    # Trigger alarms if applicable
    # チャットワークが改行コードに対応されないので改行を明示的に導入
    [ -z "$expiration" ] && MESSAGE+="
[$host $v] UNKNOWN - Domain doesn't exist or no WHOIS server available."
    [ $expdays -lt 0 ] && MESSAGE+="
[$host $v]CRITICAL - Domain expired on $expiration $STATE_CRITICAL"
    [ $expdays -lt $critical ] && MESSAGE+="
[$host $v] CRITICAL - Domain will expire in $expdays days $STATE_CRITICAL"
    [ $expdays -lt $warning ]&& MESSAGE+="
[$host $v] WARNING - Domain will expire in $expdays days $STATE_CRITICAL"
    if [ "${TLDTYPE}" == "net" -o "${TLDTYPE}" == "com" ]; then
        # gTLDドメイン（.com .net .org .info .biz .tokyo .mobi）はOKなら正常
        # https://help.sakura.ad.jp/360000124101/#02-01
        active=`$whois $host $v | awk '/Domain Status:/ {print $3}' |cut -d':' -f2 | grep -e "ok"`
        [ -z $active ] && MESSAGE+="
[$host $v] WARNING - Domain is not active Please check ASAP $STATE_CRITICAL"
    elif [ "${TLDTYPE}" == "jp" ]; then
        # 汎用JPドメインの場合 .jpのこと
        # ActiveならOK https://help.sakura.ad.jp/360000124101/#02-01
        active=`$whois $host $v | awk '/Status/ {print $2}' |cut -d':' -f2 | grep -e "Active"`
        [ -z $active ] && MESSAGE+="
[$host $v] WARNING - Domain is not active Please check ASAP $STATE_CRITICAL"
    elif [ "${TLDTYPE}" == "tv" ]; then
        # .tvはstatusがACTIVEらしいがokが表示されるものもあるので対応
        # https://www.eurodns.com/whois-search/tv-domain-name
        active=`$whois $host $v | awk '/Status/ {print $3}' |cut -d':' -f2 | grep -e "ok"`
        [ -z $active ] && active=`$whois $host $v | awk '/Status/ {print $3}' |cut -d':' -f2 | grep -e "ACTIVE"`
        [ -z $active ] && MESSAGE+="
[$host $v] WARNING - Domain is not active Please check ASAP $STATE_CRITICAL"
    elif [ "${TLDTYPE}" == "co.jp" -o "${TLDTYPE}" == "or.jp" ]; then
        # .Stateが"Connected"ならOK
        # https://jprs.jp/about/dom-search/jprs-whois/whois-guide-view.html#4f
        active=`$whois $host $v | awk '/State/ {print $2}' |cut -d':' -f2 | grep -e "Connected"`
        [ -z $active ] && MESSAGE+="
$host $v WARNING - Domain is not active Please check ASAP $STATE_CRITICAL"
    fi
    if [ -n "$MESSAGE" -a ${count} == ${MAXCOUNT} ];then
        echo 'END'
        curl -X POST -H "X-ChatWorkToken: xxxxxx" -d "body=[info][title] $TITLE [/title] [toall]$MESSAGE [/info]" "https://api.chatwork.com/v2/rooms/xxxxx/messages"
        exit
    fi
    # No alarms? Ok, everything is right.
    echo "OK - Domain will expire in $expdays days"
done
```

# 結果

![スクリーンショット 2021-11-18 4.07.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/555632/cfe735b5-7551-0ebb-5368-2bffaf660dc4.png)

# 最後に

読んでいただきありがとうございます。
今回の記事はいかがでしたか？
・こういう記事が読みたい
・こういうところが良かった
・こうした方が良いのではないか
などなど、率直なご意見を募集しております。
