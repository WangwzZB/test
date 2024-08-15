# 使用GithubAction转换md文档为知乎格式并生成Gitee在线图像链接

> [md2zhihu github](https://github.com/drmingdrmer/md2zhihu/blob/main/README-cn.md)
>
> [用markdown写知乎文章的完美解决方案](https://blog.openacid.com/toolkit/md2zhihu/)
>
> 

## 1.创建github仓库

![image-20240815220628710](images/image-20240815220628710.png)

选择创建public仓库，私有仓库github action功能受限

![image-20240815220750215](images/image-20240815220750215.png)

该仓库只有一个分支main

![image-20240815231237114](images/image-20240815231237114.png)



## 2. 创建gitee仓库做在线图片的存储并生成私人Token

- 创建仓库

![image-20240815223828818](images/image-20240815223828818.png)

- 点击克隆/下载，保存仓库地址:`https://gitee.com/<ower拥有者>/<repo仓库名>.git`, 使用私人令牌时的用户名

![image-20240815224029886](images/image-20240815224029886.png)

![image-20240815224203611](images/image-20240815224203611.png)

- 创建私人令牌
  - 进入 gitee 主页，点击右上角的个人图标，然后选择弹出菜单的设置选项
  - 选择左侧操作栏的私人令牌一项
  - 点击生成新令牌，然后输入令牌名称，生成 token并及时保存
  ![image-20240815224457589](images/image-20240815224457589.png)
  

![image-20240815224516463](images/image-20240815224516463.png)



## 3.设置Github

- 设置仓库属性允许github action推送代码
  - settings----> Actions----> General---->Workflow permissions------>勾选Read and write permissions和Allow GitHub Actions to create and approve pull requests--->save


![image-20240815223438822](images/image-20240815223438822.png)




- 进入Action配置工作流yml文件

![image-20240815220858273](images/image-20240815220858273.png)



- 将yml命名为md2zhihu.yml, 设置gitee仓库地址: 按照本文第2小节保存的内容替换`asset_repo`地址的拥有者名字和仓库名字， 文件内容如下:

```bash
name: md2zhihu
on: [push]
jobs:
  md2zhihu:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: drmingdrmer/md2zhihu@main
      env:
        GITHUB_USERNAME: ${{ github.repository_owner }}
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        pattern: >
            _posts/*.md
            _posts/*.markdown

        asset_repo: https://${{secrets.GITEE_PUSH_PIC_REPO}}@gitee.com/<ower拥有者>/<repo仓库名>.git
```

![image-20240815225421403](images/image-20240815225421403.png)

推送代码后， `md2zhihu action`将创建一个新分支 `master-md2zhihu`， 转换项目`_posts/` 目录中的 Markdown 文件，并将它们保存在 `_md2zhihu` 目录中。

- 提交workflow yml文件修改

![image-20240815221726927](images/image-20240815221726927.png)

- 设置变量`secrets.GITEE_PUSH_PIC_REPO`为gitee仓库私人令牌

  - 添加actions 仓库 secrets， 设置环境变量

  ![image-20240815230026110](images/image-20240815230026110.png)

![image-20240815230209214](images/image-20240815230209214.png)



## 3. 创建本地仓库添加目的转换文件并推送本地仓库到远端

- 查看github账户的邮箱

![image-20240815232425216](images/image-20240815232425216.png)

- 配置ssh密钥，保持和github的安全连接

  -  `ls -al ~/.ssh`检查输出中是否存在以下文件：id_rsa（私钥）id_rsa.pub（公钥）

  -  如果`id_rsa.pub`文件存在，` cat ~/.ssh/id_rsa.pub`复制公钥

  -  如果不存在： ①`cd ~/.ssh //进入用户主目录`,②` ssh-keygen -t rsa -C "youremail@example.com"`, ③`cat id_rsa.pub`复制公钥

  -  登录github 右上角账户设置----> SSH and GPG keys---->new ssh key---->添加刚复制的密钥并起一个title 用以标识你的设备（随便命名）

  ![image-20240815232647541](images/image-20240815232647541.png)

  ![image-20240815232741053](images/image-20240815232741053.png)

- 创建一个本地文件夹（本地项目），专门用于转化md文章为知乎格式，并初始化`git init`(windows可下载 git bash命令行软件)

- 与远程github仓库建立关联: `$ git remote add origin git@github.com:[你的用户名称]/[你的仓库名称].git`

- 切换 分支 到Main `git branch -M main`

- 拉取远程main分支与本地main分支合并`git pull origin main:main`

- 在本地仓库文件夹下创建`_posts`文件夹，并将想要转换的md文件及图片文件夹放入其中

  

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDTy/3Oqj9nAga/IFW/OtvqJbT0p0ZH2jVOWFPwMAVyXh9fLGVSi/TnLD8WGdWJpTvbrt4ZowuleXKnmB6cf/Ma/CfmhhKCxDMZX5eiE3VvvSFV0gt7PcXgXetnNl0lfB/aEsWpDI6VLvmOoeRdb5cZJYLh04Dr+dCRb9TLRMTsNVt6JJWcykns1gadi5kx/6yxkCSvQzl5U4RAON91TRgDEEf7W4GG2QXGiJlTNJLw6McTjiTqTxokYVD/pDxCGH1IEsHE/+AwRi/8JGIltRcDl2eXNfLa3I5a6MbwHVBmVC+/COp9zX0MBk62vRJ6M3G7lyEqJEcfjX5qLuMmy7Q1hPaC+kPAhwAr13G+1uA565TB5Zt5SEQsJBzBO3ewiQ9DqSjJj1HC/YYXvjtTpBY76AeRcXpUXiACB/ZaawrCdyfZR0SaAFXkluB51EytSeWXQCGX6hXIuL3uyhoZ7Sq6pHiSQeLRGo5jZXyfoyF2s4IeOJe2hLTRtFGqvdWjFQM= 2642509545@qq.com

## 4. 获取生成后文件

