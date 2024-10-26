
# 1 sudo 

Das Kommando sudo haben Sie schon zur Installation von Softwarepaketen verwendet. Es ermöglicht, ein einzelnes Kommando als root (oder ein anderer User) auszuführen. Sie müssen dazu nicht das Passwort von root eingeben, sondern Ihr eigenes.


`/etc/sudoers` 文件 
`/etc/sudoers` enthält detaillierte Regeln, ==welcher Benutzer/welche Gruppe `sudo` für welches Kommando nutzen kann. ==
Darin kann auch festgelegt werden, dass für `sudo` in manchen Situationen kein Passwort eingegeben werden muss



`sudo -i`
As Kommando sudo -i (für interactive) öffnet eine Shell als root. 



查询资料 用到的man命令 
man 8 sudo # Shows the sudo command documentation. 
man 5 sudoers # Shows the sudoers configuration file documentation.


# 2 su
切换到 root user ,  执行完了 su 之后, 还需要输入 root 的 password  


mit dem Kommando su können Sie eine Shell als root öffnen, indem Sie das Passwort von root eingeben



