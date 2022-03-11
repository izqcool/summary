### 如何在一台电脑上配置多个SSH key  

大多数时候一个SSH key可以满足我们日常开发，但是偶尔也会遇到一台电脑需要配置多个SSH Key的场景(例如：公司的项目是gitlab, 个人的项目是github),需要区分SSH Key, 分别配置。

---

假设你已经生成了公司的公共 SSH Key（绑定了gitlab公钥）, 现在添加自己个人SSH Key

执行 <font color="#fc5531">cd ~/.ssh</font> 到 .ssh目录下查看是否已经生成SSH Key, 如果已经生成，一般在.ssh目录下会出现**id_rsa**,**id_rsa.pub**,**knowsn_hosts** 3个文件，执行 <font color="#fc5531">cat id_rsa.pub</font> 查看当前已经生成的SSH Key

![查看已生成的SSH Key](https://thumbnail1.baidupcs.com/thumbnail/4b4c7ce03kf950bcae307d7bc4457437?fid=3157174925-250528-709664591135161&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-H2OFsdNQE%2bKoIzTh1cFNX7xaLPY%3d&expires=8h&chkbd=0&chkv=0&dp-logid=458443100622596981&dp-callid=0&time=1646967600&size=c1920_u1080&quality=90&vuk=3157174925&ft=image&autopolicy=1)

执行 <font color="#fc5531">ssh-keygen -t rsa -C "youremail@example.com" -f ~/.ssh/github_rsa</font>, 注意这里使用自己的email, **github_rsa** 为你为github生成的rsa的别名，可随意命名。

![生成新的SSH Key并命名](https://thumbnail1.baidupcs.com/thumbnail/ebe6729dcj59871cd5be994489b5cce3?fid=3157174925-250528-268415826343568&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-O0RO37kF72nUqxtmMzSaEUoMvIw%3d&expires=8h&chkbd=0&chkv=0&dp-logid=460949840899372098&dp-callid=0&time=1646978400&size=c1920_u1080&quality=90&vuk=3157174925&ft=image&autopolicy=1)

将新生成的SSH Key添加到github
![SSH Key添加到github](https://thumbnail1.baidupcs.com/thumbnail/ea1e316c3k1c75fa8e548ff50b06d332?fid=3157174925-250528-1084903354585938&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-%2f6%2bh8jyCuqa46yak24PTRPdu1KU%3d&expires=8h&chkbd=0&chkv=0&dp-logid=461015152733783220&dp-callid=0&time=1646978400&size=c1920_u1080&quality=90&vuk=3157174925&ft=image&autopolicy=1)

现在我们已经将公钥添加到了gitlab和github中，接下来我们需要将私钥添加到本地：

分别执行 <font color="#fc5531">ssh-add id_rsa</font> 和 <font color="#fc5531">ssh-add github_rsa</font>

![添加私钥到本地](https://thumbnail1.baidupcs.com/thumbnail/68b413701re27cf9542809737ff7b1ee?fid=3157174925-250528-417157513611281&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-eSjwTZ47IVhla6HPMqmO3VwO4No%3d&expires=8h&chkbd=0&chkv=0&dp-logid=461116852859763819&dp-callid=0&time=1646978400&size=c1920_u1080&quality=90&vuk=3157174925&ft=image&autopolicy=1)


执行 <font color="#fc5531">ssh-add -l</font> 查看已经添加的本地私钥,验证是否添加成功
![查看本地私钥](https://thumbnail1.baidupcs.com/thumbnail/4dfde37f9u86f2fed7a682c4831c4605?fid=3157174925-250528-502054411994492&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-auon4T%2bUuNUFKiTjllTz6czYowc%3d&expires=8h&chkbd=0&chkv=0&dp-logid=461162248716229008&dp-callid=0&time=1646978400&size=c1920_u1080&quality=90&vuk=3157174925&ft=image&autopolicy=1)
我们可以看到，此时本地的两个私钥对应的邮箱分别为zuoqiang@rockontrol.com(公司个人邮箱)和584942555@qq.com(自己邮箱)

通过以上步骤，公钥和私钥分别别添加到了git服务器和本地了，接下来我们需要创建一个本地的秘钥配置文件，来实现根据仓库的remote链接地址来自动选择合适的私钥：

在ssh目录下执行 <font color="#fc5531">vim config</font>
![](https://thumbnail1.baidupcs.com/thumbnail/ca7e25be3kc4146c38d69212e5c19e8d?fid=3157174925-250528-359485424458442&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-NBEaEWfURFPjMzrlMQvdXbXVQ8g%3d&expires=8h&chkbd=0&chkv=0&dp-logid=461315973925040334&dp-callid=0&time=1646978400&size=c1920_u1080&quality=90&vuk=3157174925&ft=image&autopolicy=1)
编辑如下内容：
```
Host github
HostName github.com
User zuoqiang
IdentityFile ~/.ssh/github_rsa

Host gitlab
HostName git.querycap.com
User zuoqiang
IdentityFile ~/.ssh/id_rsa 
```
该文件分为多个用户的配置，每个配置项具体如下：
* **Host** 仓库网站的别名，可随意取
* **HostName** 仓库网站的域名（也可以是ip）
* **User** 仓库网站上的用户名
* **IdentityFile** 私钥的绝对路径

为了确保配置正确，可以使用 <font color="#fc5531">ssh -T</font> 命令检测配置的Host是否是连通
![验证是否联通](https://thumbnail1.baidupcs.com/thumbnail/d67cdd9b9r1df9468df8bc9b454c0b73?fid=3157174925-250528-69669243190116&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-3TfMoJsJajP2%2boyAKnhzWH7iwFs%3d&expires=8h&chkbd=0&chkv=0&dp-logid=461609227399680879&dp-callid=0&time=1646978400&size=c1920_u1080&quality=90&vuk=3157174925&ft=image&autopolicy=1)

此时如果想使用github仓库的话，我们只需要给该仓库配置用户名信息，进入仓库后执行：
```
git config --local user.name "xxx"
git config --local user.email "584942555@qq.com"
```

git的配置分为3级： System-->Global-->Local。 System为系统级别，Global为全局配置（一般我们的电脑上是设置公司为全局的）, Local为仓库级。 执行优先级为 Local>Global>System,所以只需要我们自己的项目的仓库更改local config即可。








