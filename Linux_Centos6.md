## CentOS 6

### OS的設定のメモなど

### Mail関連

#### defaultのMTA(メールサーバー)の確認

alternatives --display mta

```
mta -ステータスは自動です。
リンクは現在 /usr/sbin/sendmail.postfix を指しています。
/usr/sbin/sendmail.postfix - 優先項目 30
 スレーブ mta-pam: /etc/pam.d/smtp.postfix
 スレーブ mta-mailq: /usr/bin/mailq.postfix
 スレーブ mta-newaliases: /usr/bin/newaliases.postfix
 スレーブ mta-rmail: /usr/bin/rmail.postfix
 スレーブ mta-sendmail: /usr/lib/sendmail.postfix
 スレーブ mta-mailqman: /usr/share/man/man1/mailq.postfix.1.gz
 スレーブ mta-newaliasesman: /usr/share/man/man1/newaliases.postfix.1.gz
 スレーブ mta-aliasesman: /usr/share/man/man5/aliases.postfix.5.gz
 スレーブ mta-sendmailman: /usr/share/man/man1/sendmail.postfix.1.gz
現在の「最適」バージョンは /usr/sbin/sendmail.postfix です。
```
わかりずらいが、デフォルトはPostfixらしい

### MTAの変更
update-alternatives --config mta


