之前所有项目每个环境的构建都对应 jenkins 上的一个任务，每次在 gitlab 上 push 代码之后，都需要手动在 jenkins 上点个部署。这显然比较麻烦。刚好最近了解到 jenkins 有构建触发器这样的功能，可以利用 gitlab webhook 来触发 jenkins 任务。

具体做法如下：

1、打开 jenkins 任务配置页面，点击构建触发器tab

![image](//qhyxpicoss.kujiale.com/2018/03/22/LKZ23RAKAIB4ARYEAAAAABI8_1966x1198.png)

勾选所需要的触发条件，我选择在push操作发生的时候进行构建。

GitLab CI Service URL 后面是触发这个构建操作 api 调用的地址，这个 URL 待会儿是需要用到的。

配置完 jenkins 之后，需要在 gitlab 里面配置 webhook

打开 gitlab project ，点击 Settings ， 接着点击 Integrations 。

![image]()

第一行 URL 填写的就是 jenkins 构建触发器配置页面 GitLab CI Service URL 后面的 URL 地址。默认勾选的 hook 是 push event 。

点击最下面的 Add webhook ，任务似乎就这样完成了。

于是我点击了 webhook 旁边的 test 按钮，测试一下对 jenkins 的调用是否生效。果然，接口报了401的错误。

![image](//qhyxpicoss.kujiale.com/2018/03/23/LKZ23RAKAIB4ARYEAAAAABY8_1934x54.png)

看到 invalid token 突然想到刚才在 webhook 创建的时候有个 secrect token 没有填写，这个 token 应该就是 jenkins 那边提供进行 api 权限认真的。再回到 jenkins 构建触发器配置的页面，在一个不起眼的角落里发现了一个高级选项的按钮。

![image](//qhyxpicoss.kujiale.com/2018/03/23/LKZ23RAKAIB4ARYEAAAAABQ8_2140x1182.png)

点开之后发现有生成 secrect token 的功能。

![image](//qhyxpicoss.kujiale.com/2018/03/23/LKZZROQKAIB36X4SAAAAADY8_1808x592.png)

勾选所需配置之后，点击 generate 按钮生成 secrect token 。

将这里生成的 secret token 加入 webhook 的配置中，点击报错，再次尝试，这次成功地触发了 jenkins 任务的构建，大功告成！