# 通过https在ec2/vm/bare metal上建立Blobber

## 准备工作

ec2 / vm / bare metal上已安装docker

## 初始设置
### 设置Blobbers目录
在git/blobber中运行以下命令

    $ ./docker.local/bin/blobber.init.setup.sh

## 建立和启动节点

1. 为每个节点的容器建立一个网络，名为testnet0，使容器之间可相互通信

        $ docker network create --driver=bridge --subnet=198.18.0.0/15 --gateway=198.18.0.255 testnet0

2. 根据您的config/0chain_blobber/validator.yaml和config/0chain_blobber/0chain_blobber.yaml 配置文件中更新block_worker的url。
例如：如果您想连接到five network，则设置：

        block_worker: https://five.devnet-0chain.net/dns
3. 编辑docker.local/p0docker-compose.yml文件，并根据您的域名对命令行中的< localhost >进行替换

4. 进入blobber1目录(git/blobber/docker.local/blobber1)，并通过以下命令运行容器：

        $ ../bin/p0blobber.start.sh

此操作将会把blobber1加入到网络中。您可以重复第4步操作加入其它的blobber。


# Configuring Https
备注：如果您已经在nginx/haproxy中配置了ssl，则可以忽略此步骤。只需要在配置文件中添加路径并重启服务。

1. 在blobber的repo中进入https目录
        
        cd /blobber/https
2. 编辑docker-compose.yml文件，并根据您的时机邮件和域名替换<your_email>, <your_domain>。请确保根据域名供应商情况为您的域名和IP地址添加A类型的记录。 

3. 用以下命令部署nginx和certbot

        docker-compose up -d
4. 检查certbot日志，确认证书已生成。如果证书正确生成，您将会在日志文件中看到"Congratulations! Your certificate and chain have been saved at: /etc/letsencrypt/live/<your_domain>/fullchain.pem"。
        
        docker logs -f https_certbot_1 
5. 编辑/conf.d/nginx.conf文件，将在80端口配置处required locations所在行的注释取消。将所有服务器配置443端口相关行的注释取消，并将不需要的位置内容所在行进行注释。请不要忘记用您实际的域名替换<your_domain>。

6. 重启docker-compose后，您就可以通过https访问blobber了。

        docker-compose restart
