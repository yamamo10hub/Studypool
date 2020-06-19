### RTX1200

L2TP-IPsecで通信する設定
```
pp select anonymous
 pp bind tunnel33-tunnel48
 pp auth request mschap-v2
 pp auth username User001 hogefuga
 ppp ipcp ipaddress on
 ppp ipcp msext on
 ppp ipv6cp use off
 ip pp remote address pool 10.0.100.0-10.0.100.50
 ip pp tcp mss limit auto
 pp enable anonymous
tunnel select 33
 tunnel template 34-48
 tunnel encapsulation l2tp
 ipsec tunnel 33
  ipsec sa policy 33 33 esp aes-cbc sha-hmac
  ipsec ike keepalive use 33 off
  ipsec ike local address 33 10.0.0.254
  ipsec ike nat-traversal 33 on
  ipsec ike pre-shared-key 33 text jizen001
  ipsec ike remote address 33 any
 l2tp tunnel disconnect time off
 l2tp keepalive use on 10 30
 l2tp syslog on
 ip tunnel tcp mss limit auto
 tunnel enable 33
ipsec use on
ipsec transport 33 33 udp 1701
ipsec transport template 33 34-48
l2tp service on
```

### windows10側  
1. コントロールパネルのネットワーク経由ではなく、スタートメニューから歯車で設定を表示する。  
2. ネットワークとインターネット → VPN → VPN接続を追加する
	* VPNの種類(L2TP-IPsec)
	* 事前共有キー(jizen001)
	* サインイン情報(ユーザー名, パスワード)
で作成  
3. 関連設定 → アダプターのオプションを変更する
	* 「セキュリティ」→「データの暗号化」を「暗号化が必要」に指定
	* 「認証」→「次のプロトコルを許可する」で MS-CHAP v2 にチェック
4. regedit でレジストリエディタを編集する
	```
	▲HKEY_LOCAL_MACHINE
	　▲SYSTEM
	　　▲CurrentControlSet
	　　　▲Services 
	       ▲PolicyAgent
	```
「編集」→「新規」→「DWORD(32ビット)値」  
作成したエントリの名前を「AssumeUDPEncapsulationContextOnSendRule」値を「2」にする  

再起動して接続できればOK