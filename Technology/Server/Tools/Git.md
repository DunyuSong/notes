# Git
## Git使用
### SSH key的生成和配置
1. 检查本地是否已经有SSH key：ls -al ~/.ssh
2. 如果有的话应该会有id_rsa类似的文件，没有的进行步骤3，手动生成。有的话直接步骤4拷贝即可。
3. 生成SSH：打开 Git bash 输入：ssh-keygen -t rsa -b 4096 -C "shandongdongtester@gmail.com"
4. 拷贝公钥：clip < ~/.ssh/id_rsa.pub
5. 将拷贝的公钥添加到GitHub：https://github.com/settings/keys -->New SSH key --> 直接粘贴刚才拷贝的公钥。
- 参考GitHub上的帮助文档。

### 本地项目关联远程仓库
- 将本地已经存在的项目使用git来进行托管，并且提交到git仓库中

    ```shell
    git init
    git add .
    git commit -m "init commit"
    
    连接到远程仓库
    git remote add origin 你的远程仓库地址
    
    获取远程仓库中的文件
    git pull --rebase origin master
    
    将项目推送到远程仓库
    git push -u origin master
    
    
    ```

    

- 
