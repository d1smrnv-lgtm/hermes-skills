S17 / s17.ipt.local hosts ManageEngine Endpoint Central at D:\UEMS_CentralServer; web UI is https://s17:8383/ via bundled nginx. This environment can read \\S17\c and \\S17\d, but remote service control via WinRM/SCM was denied and IPC$ returned SMB error 67 (Nov 2026).
§
On this Mac (user dns), ~/Library/Containers and ~/Library/Group Containers entries are protected by macOS containermanagerd/SIP — rm (even sudo) fails with 'Operation not permitted' on the .com.apple.containermanagerd.metadata.plist. These must be deleted manually in Finder. Same applies to some Application Support folders with TCC-protected extended attributes (e.g. Microsoft Edge). System-wide /Library paths require sudo for deletion.
§
VPS VPN server: 37.27.215.108 (Ubuntu), WireGuard iface wg0 IP 10.9.9.1/24, UDP 39743. Client config: /Users/dns/Desktop/hermes-first-task/wireguard.conf. Server config: /etc/wireguard/wg0.conf. VPN subnet: 10.9.9.0/24. Server public key: VxBNWuQbXDFLC5Vu4DG9dxvphqJlGA7hmbN6qFDH/iM=. Client public key: W52q/vIRhebtuKDJvbOWDyUzs4kx3GJHNu68+IiahTk=. NAT rule: iptables -t nat -A POSTROUTING -s 10.9.9.0/24 -o eth0 -j MASQUERADE
§
NetMon dashboard: local Flask app at http://127.0.0.1:5000, files in /Users/dns/netmon/, SQLite history /Users/dns/netmon/netmon.db. Desktop shortcut /Users/dns/Desktop/NetMon.webloc.
§
Local network dashboard NetMon: Flask app at /Users/dns/netmon (app.py + templates/index.html), served on http://127.0.0.1:5000, SQLite history in netmon.db. Features: multi-host ping comparison, traceroute, HTTP check, sboy.rf keyword check, Google News RSS search for outages, 30s auto-refresh. Desktop shortcut: ~/Desktop/NetMon.webloc.
§
GitHub: аккаунт d1smrnv-lgtm, gh CLI 2.103.3 (/opt/homebrew/bin/gh), авторизован через device flow, токен в keyring (scopes: repo, workflow, read:org, gist). git user.name/user.email НЕ заданы — коммиты требуют настройки identity. Коннектора github в Hermes нет, работа идёт через gh.
