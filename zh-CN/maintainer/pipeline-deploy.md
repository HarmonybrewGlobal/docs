# 流水线部署

## 1 前言

本文档供项目维护者或有二次开发需求（需要自己部署一整套 Harmonybrew 项目）的用户用作参考，普通用户无需阅读。

本项目的 CI 系统基于华为云基础设施构建，因此文档中的资源操作及配置均以华为云为基准进行描述。动手能力强的开发者可参考其逻辑，自行迁移至其他云厂商。

接下来以二次开发的场景为例，讲解流水线的部署流程。

## 2 前置准备

### 2.1 ECS

需要准备一些服务器，承担不同的职责。

这里假设我们准备了 6 台服务器，我们分别给他们取不同的名字、分配不同的职责

| 主机名         |  职责                                                   |
| ---------------|---------------------------------------------------------|
| image-builder  | 专门用来跑 auto_images.sh，负责构建出 ci-runner 镜像    |
| webhook-filter | 专门用来跑 webhook_filter.py，负责过滤 webhook 报文     |
| ci-runner-1    | 其他流水线脚本都跑在这些机器上                          |
| ci-runner-2    | 其他流水线脚本都跑在这些机器上                          |
| ci-runner-3    | 其他流水线脚本都跑在这些机器上                          |
| ci-runner-4    | 其他流水线脚本都跑在这些机器上                          |

这些服务器需要满足以下条件：
1. 是 arm 服务器，不是 x86 服务器
2. Region 在香港或国外（CI 执行过程中需要去 GitHub 下载源码和开发工具，中国大陆的服务器无法稳定访问 GitHub）
3. 安装好 JDK，版本在 JDK 8 以上
4. 安装好 Docker

### 2.2 域名

准备一个属于你自己的域名，且最好完成 ICP 备案。这个域名后续将用来作为 CDN 加速域名。

由于华为云现在已经不卖域名，你需要去其他云厂商或专门的域名服务商购买域名。

域名准备好后，还需要给你的域名申请一份 SSL 证书。如果你是个人开发者，为节省费用，建议使用 [Let's Encrypt](https://letsencrypt.org/) 提供的免费证书。

### 2.3 OBS 和 CDN 

前往 OBS 控制台，创建一个 OBS 桶用来存储软件包，名字和 Region 随意。

前往 CDN 控制台，将你的域名配置为 CDN 加速域名，CDN 回源地址指向你的 OBS 桶。

CDN 服务范围的选择：
* 如果完成了 ICP 备案，可以选“中国大陆”或“全球”。
* 如果没完成 ICP 备案，只能选“中国大陆境外”。

CDN 加速域名配置完成之后，把你的 SSL 证书也部署到 CDN 上，以便后续使用 https 链接。

### 2.4 SWR

需要在 SWR 控制台里面创建一个组织。流水线构建出来的 Docker 镜像会上传到这个组织上。

### 2.5 代码仓

1. 在 AtomGit 上面创建一个新的组织，名字和 Region 随意。
2. 将 Harmonybrew 组织下的所有代码仓全部 fork 一份到这个新的组织下。
3. 把 `brew`、`install`、`ci` 这 3 个仓库下载到本地，进行修改，把里面的组织名字和 CDN 加速域名都替换成自己的数据。

    对 3 个仓库都执行这个操作
    ```sh
    # 假设你的组织名字为 Brewharmony、CDN 加速域名为 cdn.brewharmony.com
    find . -type f -not -path '*/.git/*' -exec sed -i 's@atomgit.com/Harmonybrew@atomgit.com/Brewharmony@g' {} +
    find . -type f -not -path '*/.git/*' -exec sed -i 's@harmonybrew.atomgit.com@cdn.brewharmony.com@g' {} +
    ```

    对 brew 仓库还需额外执行这个操作
    ```sh
    # 假设你的组织名字为 Brewharmony
    find . -type f -not -path '*/.git/*' -exec sed -i 's@Harmonybrew@Brewharmony@g' {} +
    ```

4. 把改好的代码提交回你的组织里面。

### 2.6 环境变量

准备好以下环境变量的值，接下来的部署步骤会用上。

<table>
  <thead>
    <tr>
      <th>变量名</th>
      <th>变量值获取方法</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ATOMGIT_TOKEN</td>
      <td>创建一个 AtomGit 账号作为机器人账号，将其加入你 fork 的 homebrew-core 仓库，
         成为项目成员，并赋予维护者权限。之后进入个人中心创建访问令牌，这个变量记录的就是访问令牌的值。
      </td>
    </tr>
    <tr>
      <td>HUAWEICLOUD_SDK_AK</td>
      <td>在华为云控制台右上角找到“我的凭证”，进入该页面操作生成。</td>
    </tr>
    <tr>
      <td>HUAWEICLOUD_SDK_SK</td>
      <td>同上，AK/SK 是一对的。</td>
    </tr>
    <tr>
      <td>OBS_BUCKET</td>
      <td>在上面的步骤已经创建过 OBS 桶，请把上面创建的桶名记录下来。</td>
    </tr>
    <tr>
      <td>OBS_REGION</td>
      <td>在上面的步骤已经创建过 OBS 桶，请把所属 Region 记录下来。</td>
    </tr>
    <tr>
      <td>JWS_SIGNING_KEY</td>
      <td>
        手动创建一对 RSA 密钥。
        命令：<code>openssl genrsa -out private.pem 2048 &amp;&amp; openssl rsa -in private.pem -pubout -out public.pem</code>。
        生成出来的密钥对要保管好，里面的私钥就是 JWS_SIGNING_KEY 的值，
        公钥需要放到 brew 代码仓中，覆盖代码仓中原有的 homebrew-1.pem 文件。
      </td>
    </tr>
    <tr>
      <td>JWS_SIGNING_KEY_ID</td>
      <td>固定值 homebrew-1。</td>
    </tr>
    <tr>
      <td>HOMEBREW_ATOMGIT_API_TOKEN</td>
      <td>跟上面的 ATOMGIT_TOKEN 是同一个数据，无需重复创建。</td>
    </tr>
    <tr>
      <td>HOMEBREW_GITHUB_API_TOKEN</td>
      <td>在 GitHub 上面用自己的个人账号生成一个 token。</td>
    </tr>
    <tr>
      <td>SWR_REGION</td>
      <td>在上面的步骤已经创建过 SWR 组织，请把上面创建的组织名记录下来。</td>
    </tr>
    <tr>
      <td>SWR_ORGANIZATION</td>
      <td>在上面的步骤已经创建过 SWR 组织，请把所属 Region 记录下来。</td>
    </tr>
  </tbody>
</table>

## 3 创建 CodeArts 项目

1. 进入 CodeArts 服务的工作台，点击右上角进入工作台。
2. 进入工作台后，创建新项目。名称任意，类型是 Scrum。

## 4 创建资源池

1. 在 CodeArts 服务的工作台，找到左侧菜单的 “持续交付”-“编译构建”，点进去。
2. 在“编译构建”页面的右上角找到“资源池管理”，点进去。
3. 在“资源池管理”页面，找到“新建资源池”按钮，点击它，选择“自定义资源池”。
4. 创建第一个资源池。资源池名称：`ci-runner`；类型：`LINUX-DOCKER`
5. 创建第二个资源池。资源池名称：`image-builder`；类型：`LINUX`

## 5 注册执行机
1. 进入资源池管理界面，点击资源池的名称 `ci-runner`。
2. 在 `ci-runner` 资源池的详情页面，点右上角“新建代理”。
3. 配置代理信息：
    * 上半部分除“重启免注册”以外，其他开关都不要开启；
    * 下半部分录入你提前准备好的 AK/SK，代理名称填 `ci-runner-1`，代理工作空间填 `/opt/ci-runner-1`
4. 配置完成后，点击“生成命令”、“复制命令”，生成的命令粘贴到服务器中执行。
5. 如果有多台执行机，就重复执行 3、4 步骤。执行机命名规则：`ci-runner-1`、`ci-runner-2` ...
6. 进入 `image-builder` 资源池的详情页面，同样执行 3、4 步骤，涉及到名字的地方都命名为 `image-builder`。这个资源池不常用，只注册一台执行机即可。

## 6 创建编译构建任务

1. 进入“编译构建”界面，新建一个任务
    * 名称：`auto-bottle 任务`
    * 代码源：来自流水线
    * 使用空白构建模板
    * 选择 `ci-runner` 资源池
    * 点击左边“添加构建步骤”，选“容器类”-“使用SWR公共镜像”
    * 构建步骤添加出来后，填写信息。
    	* 名称：随意
        * 镜像地址：根据你注册的 SWR 组织信息构造出 ci-runner 镜像的下载地址。假设你的组织名字为 brewharmony、Region 在北京四，那你的镜像下载地址就是这样：`swr.cn-north-4.myhuaweicloud.com/brewharmony/ci-runner:latest`
        * 命令：`python3 src/auto_bottle.py`
2. 以同样的操作，将 `auto-merge`、`auto-bump`、`auto-static`、`auto-stats` 这几个任务也都创建出来。这些任务除了脚本名字不一样以外其他配置完全相同。
3. 进入“编译构建”界面，新建一个任务
    * 名称：`auto-image 任务`
    * 代码源：来自流水线
    * 使用空白构建模板
    * 选择 `image-builder` 资源池
    * 点击左边“添加构建步骤”，选“所有步骤”-“执行shell命令”
    * 构建步骤添加出来后，填写信息。名称：随意；命令：`sh src/auto_image.sh`


## 7 配置编译构建任务参数

对上面创建出来的所有编译构建任务，全部进行一遍参数设置。

操作方法：进入“编译构建”界面，点击任务名字进入详情页面，点击右上角“参数设置”，录入自定义参数。


各任务涉及到的参数如下，所需参数在最开始的“前置准备”已经准备好，直接录入即可：

<table>
  <tr>
    <th>任务</th>
    <th>需要录入的参数</th>
  </tr>

  <tr>
    <td>auto-bottle 任务</td>
    <td>
      <ul>
        <li>ATOMGIT_TOKEN</li>
        <li>HUAWEICLOUD_SDK_AK</li>
        <li>HUAWEICLOUD_SDK_SK</li>
        <li>OBS_BUCKET</li>
        <li>OBS_REGION</li>
        <li>WEBHOOK_PAYLOAD（不用填默认值，开启“运行时设置”即可）</li>
        <li>PIPELINE_ID（不用填默认值，开启“运行时设置”即可）</li>
        <li>PIPELINE_RUN_ID（不用填默认值，开启“运行时设置”即可）</li>
        <li>PIPELINE_NUMBER（不用填默认值，开启“运行时设置”即可）</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-merge 任务</td>
    <td>
      <ul>
        <li>ATOMGIT_TOKEN</li>
        <li>HUAWEICLOUD_SDK_AK</li>
        <li>HUAWEICLOUD_SDK_SK</li>
        <li>OBS_BUCKET</li>
        <li>OBS_REGION</li>
        <li>JWS_SIGNING_KEY</li>
        <li>JWS_SIGNING_KEY_ID</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-bump 任务</td>
    <td>
      <ul>
        <li>HOMEBREW_ATOMGIT_API_TOKEN</li>
        <li>HOMEBREW_GITHUB_API_TOKEN</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-static 任务</td>
    <td>
      <ul>
        <li>HUAWEICLOUD_SDK_AK</li>
        <li>HUAWEICLOUD_SDK_SK</li>
        <li>OBS_BUCKET</li>
        <li>OBS_REGION</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-stats 任务</td>
    <td>
      <ul>
        <li>HUAWEICLOUD_SDK_AK</li>
        <li>HUAWEICLOUD_SDK_SK</li>
        <li>OBS_BUCKET</li>
        <li>OBS_REGION</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-image 任务</td>
    <td>
      <ul>
        <li>SWR_REGION</li>
        <li>SWR_ORGANIZATION</li>
      </ul>
    </td>
  </tr>
</table>

## 8 新建服务扩展点
1. 进入 CodeArts 服务的工作台，找到“通用设置”-“服务扩展点管理”，点击“新建服务扩展点”。类型选 GitCode（CodeArts 平台现在还不认识 GitCode 的新名字，只能用旧名字）；连接名称是自定义的，这里填 harmonybrew；token 填你注册的那个 AtomGit 机器人账号的 token。

## 9 创建流水线
1. 在 CodeArts 服务的工作台，找到左侧菜单的 “持续交付”-“流水线”，点进去，创建一条流水线。
    * 名称：auto-bottle
    * 流水线源：AtomGit
    * 服务扩展点：harmonybrew
    * 代码仓：ci（指向你 fork 的 ci 仓库）
    * 默认分支：main
    * 选择模板：空模板
2. 编辑 auto-bottle 流水线，点击上方菜单“任务编排”，点击“阶段\_1”里面的“新建任务”。
    * 选择“Build构建”
    * 名称：auto-bottle
    * 任务：auto-bottle 任务
    * 仓库：ci

按照相同的方法，将 `auto-merge`、`auto-bump`、`auto-static`、`auto-stats`、`auto-image` 这几条流水线创建出来。他们之间的区别只有触发设置不同，前面的创建步骤是一样的。

流水线创建完成后，依次修改各个流水线的触发设置：

<table>
  <tr>
    <th>流水线</th>
    <th>触发设置</th>
  </tr>

  <tr>
    <td>auto-bottle</td>
    <td>
      <ul>
        <li>启用 Webhook，关闭 IAM 认证。</li>
        <li>开启“并发策略”，把并发个数设置得跟你的执行机数量一致。超过并发后执行策略是“排队等待”</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-merge</td>
    <td>
      <ul>
        <li>定时任务， corn 表达式是 `0 0/10 * * * ?`（每 10 分钟执行一次）</li>
        <li>开启“并发策略”，并发个数固定为1。超过并发后执行策略是“忽略不执行”</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-bump</td>
    <td>
      <ul>
        <li>定时任务， corn 表达式是 `0 0 6 * * ?`（每天 6:00 执行一次）</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-static</td>
    <td>
      <ul>
        <li>不开启任何触发设置（也就是仅允许手动触发）</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-stats</td>
    <td>
      <ul>
        <li>定时任务， corn 表达式是 `0 0 0 * * ?`（每天 0:00 执行一次）</li>
      </ul>
    </td>
  </tr>

  <tr>
    <td>auto-image</td>
    <td>
      <ul>
        <li>不开启任何触发设置（也就是仅允许手动触发）</li>
      </ul>
    </td>
  </tr>
</table>

## 10 搭建 webhook 过滤服务器

1. 登陆 `webhook-filter` 服务器，直接在宿主机上安装 nginx、python 和所需的 pip 三方库。
    ```sh
    apt update
    apt install -y nginx python3 python3-requests python3-fastapi python3-uvicorn
    ```
2. 创建 nginx 配置文件 `/etc/nginx/sites-available/webhook_filter.conf`

    ```
    server {
        listen 1234;            # 改成你实际想用的端口，配完后记得在 ECS 的安全组里面把对应端口放开
        server_name 1.2.3.4;    # 改成你实际的公网 IP

        location /v5/ {
            proxy_pass http://127.0.0.1:8090;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # 建议做其他安全加固配置，例如 IP 白名单过滤等，这里不细讲
        }

        # 其他路径返回 403
        location / {
            return 403;
        }
    }
    ```
3. 启用 nginx 服务
    ```sh
    # 创建软链接到 sites-enabled目录中
    ln -s /etc/nginx/sites-available/webhook_filter.conf /etc/nginx/sites-enabled/

    # 测试配置是否正确
    nginx -t

    # 重新加载 nginx 配置
    systemctl reload nginx
    ```
4. 在服务器上面的 /opt 目录放置 `webhook-filter.py`
    ```sh
    # 创建专用用户，命名为 filter
    adduser --system --group --no-create-home filter
    mkdir -p /opt/webhook_filter
    curl -fsSL -o /opt/webhook_filter/webhook_filter.py https://raw.atomgit.com/Harmonybrew/ci/raw/main/src/webhook_filter.py
    chown -R filter:filter /opt/webhook_filter
    ```
5. 创建 systemd 配置文件 `/etc/systemd/system/webhook-filter.service`
    ```sh
    [Unit]
    Description=AtomGit Webhook Filter for CodeArts
    After=network.target

    [Service]
    Type=simple
    # 以 filter 用户权限运行
    User=filter
    WorkingDirectory=/opt/webhook_filter
    # 启动命令
    ExecStart=/usr/bin/python3 /opt/webhook_filter/webhook_filter.py
    # 崩溃自动重启
    Restart=on-failure
    RestartSec=10
    # 日志走 journald
    StandardOutput=journal
    StandardError=journal
    SyslogIdentifier=webhook-filter

    [Install]
    WantedBy=multi-user.target
    ```
6. 启动 systemd 服务
    ```sh
    # 重载 systemd 配置
    systemctl daemon-reload
    # 启用开机自启
    systemctl enable webhook-filter.service
    # 立即启动
    systemctl start webhook-filter.service
    ```

## 11 对接 webhook
1. 在 CodeArts 服务的工作台，找到左侧菜单的 “持续交付”-“流水线”，点进去。
2. 点开 `auto-bottle` 这条流水线，点上方菜单“触发设置”，复制 webhoook 触发 URL。记录下来，并把 URL 里面的域名替换成 `<过滤服务器的公网 IP>:<过滤服务器的端口>`，协议从 https 换成 http。这个改好的 URL 后面会用到。
3. 进入 homebrew-core 仓库的 webhook 配置页面，点右上角“新建 WebHook”。
4. 把你改造过的 URL 填进去，WebhooK 密码留空，勾选“Pull Request 事件”和“激活”，最后保存。

## 12 构建第一个 ci-runner 镜像

构建第一个镜像之前需要在执行机上面进行一次手工运维操作：
1. 打开 SWR 控制台，点右上角“客户端上传”。
2. 生成登陆指令。
3. 把 AK/SK 填进去，生成长期有效指令。
4. 把指令粘贴到 image-builder 这台服务器里面执行.
5. 进入 CodeArts 页面，手动触发一次 auto-image 流水线。构建完成后，可以在 SWR 里面看到镜像了，不过第一次构建出来的镜像默认是私有的。
6. 在 SWR 里面，点开镜像的详情页面，点击右上角的“...”，然后“编辑”，设置成公开。

## 13 调整 CDN 缓存配置

华为云 CDN 默认会把文件缓存，有效期是 90 天，文件一旦缓存起来它就不会再去 OBS 拿新文件了。

在我们的 OBS 桶中，有一类文件是包索引文件，这些文件需要频繁更新，所以要手动调整缓存配置。

在 CDN 控制台里面找到“缓存配置”进行配置，缓存规则如下：
* /api：包索引缓存 10 分钟
* 默认规则：默认值 30 天不做改动

## 14 部署完成

部署完成，可以开始正常使用流水线了。
