<div style='background: -webkit-linear-gradient(top, transparent 19px, #ececec 20px), -webkit-linear-gradient(left, transparent 19px, #ececec 20px);
            background-size: 20px 20px;'>

# 前端整合多个框架项目为一

**当你有两及多个不同框架项目需要合并成一个并且需要相互跳转，实现项目多合一，共享token以及用户信息的时候，我们需要以下两种方法**

1. 使用 <b style="background: red">iframe</b> 标签进行嵌套。最简单，把两个项目部署在不同服务器上，切换 <b style="background: red">url</b>  或者子父嵌套即可。
2. 使用 <b style="background: red">Nginx</b> 转发页面，变成 "多页面" 应用。难度中等，需要在外搭建一层，然后配置 <b style="background: red">Nginx</b> 转发 。

**这篇帖子着重对第二种方法的实现展开讲解。**

<h3 style="border: 1px solid #000">一、搭建最外层的框架。</h3>
1. 新建文件夹，将两个项目放进去。目录嵌套如下图，两个项目可以是任何框架编写，vue、react、angular……。这里我们有三个a、b、c三个项目。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d56f84fcf56c47f78ebb4d5542c53f15~tplv-k3u1fbpfcp-watermark.image)

2. 新建 **package.json** 文件，方便使用命令来操作。**package.json** 定义了这个项目的各种配置信息以及依赖模块，则 **scripts** 定义了各种脚本，每一个属性对应一个脚本。在阮一峰大神写的文章里我们可以知道，不光可以使用 **npm** 脚本，比如当我们使用 **Linux** 系统的时候可以把 **shell** 脚本放在 **scripts** 里。**npm** 脚本的退出码遵循 **shell** 脚本的退出码，不为 **0** 的时候一律视为失败。执行顺序、通配符之类的我们不过多赘述，详情请看 [npm scripts 使用指南 by阮一峰。](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)

3. 我们在 **package.json** 文件里写了三个命令，对于项目的安装、启动、打包。注意所有 **:all** 的执行顺序。我们着重说一下 **concurrently** ，**concurrently** 是一个可以同时开启多个端口的 **node** 包，比较少见因为我们一般执行一段脚本命令都是单个单个去执行，因为我们要同时打包三个项目，所以使用 **concurrently** 这个 **node** 依赖包。


```json
{
    "name": "micro-examples",
    "scripts": {
      "install:all": "yarn install:a-project && yarn install:a-project && npm run install:a-project ", // 安装所有项目依赖
      "install:a-project": "cd a-project && yarn", // 三种安装依赖方式，有人喜欢用 yarn 有人喜欢用 npm，进入项目文件夹进行 install 
      "install:b-project": "cd b-project && npm install", 
      "install:c-project": "cd c-project && npm i", // 相当于 npm install  

      "serve:all": "concurrently \"serve:a-project\"  \"yarn serve:b-project\" \"yarn serve:c-project\"", // 本地启动三个项目
      "serve:a-project": "cd a-project && yarn serve",
      "serve:b-project": "cd b-project && yarn serve",
      "serve:c-project": "cd c-project && yarn serve",

      "build:all": "concurrently \"yarn build:a-project\" \"yarn build:b-project\" \"yarn build:c-project\"", // 打包三个项目
      "build:a-project": "cd a-project && yarn build",
      "build:b-project": "cd b-project && yarn build",
      "build:c-project": "cd c-project && yarn build",
    },
    "devDependencies": {
      "concurrently": "^6.0.0"
    }
}
```

<h3 style="border: 1px solid #000">二、编写 .gitlab-ci.yml，我们使用 gitlab ci来构建。</h3>

对于这块不懂的同学可以请教一下运维同学，yaml的语法。

```yml
# 指定 docker 镜像
image: a/b/node:8000

# 定义场景执行阶段，元素的顺序决定了任务执行的顺讯
stages:
  # 创建文件夹 安装最外层依赖
  - create_dir 
  # 打包所有项目
  - node_build_all
  # 构建 docker 镜像
  - docker_build_dev
  # 清除缓存
  - clean-cache  

# 任务执行前的命令
before_script:
  - *auto_devops


create_dir:
 stage: create_dir
 script:
   #  根目录下创建文件夹
   - mkdir -p 
   #  复制最外层文件夹到创建好的根目录下
   - cp -r 
   - ls 
   #  安装最外层依赖 concurrently
   - npm install

node_build_all:
  stage: node_build_all
  script:
    - npm run build:all
    - ls 
    - cp -r 
    - ls 
  when: manual

# 相当于公用函数，有重复执行的任务时调用 相当于定义了一个名为 docker_build 的函数
.docker_build: &docker_build  
  image: 指定 docker 镜像
  script:
    - docker_build
    - chart_build
  when: manual

# 构建 docker 镜像到开发环境
docker_build_dev:
  <<: *docker_build  # 合并 docker_build 函数
  stage: docker_build_dev

# 缓存
clean-cache:
  stage: clean-cache
  script:
    - rm -rf /cache/
    - clean_cache //此函数在devops.sh中
  when: manual


.auto_devops: &auto_devops |      
    # 设置 npm 镜像源
    function set_registry(){
        npm config set registry https://registry.npm.taobao.org
    }

    # 构建 docker
    function docker_build(){
        export 
        cp -rf  
        docker build -t 
        docker login -u 
        docker push 
    }
```
<h3 style="border: 1px solid #000">三、编写 default.conf，用来转发页面。</h3>


```conf
server{
        listen 90 default_server;
        server_name localhost;
        charset utf-8;
        access_log /var/log/nginx/nginx.log;
        error_log /var/log/nginx/nginx.error.log debug;

        location /a-project { // 转发到 a项目
                root /usr/share/nginx/html/a-project;
                index index.html;
                try_files $uri $uri/ /index.html;
        }
  
        location /b-project { // 转发到 b项目
                root /usr/share/nginx/html/b-project;
                index index.html;
                try_files $uri $uri/ /index.html;
        }
  
        location /a-project { // 转发到 c项目
                root /usr/share/nginx/html/c-project;
                index index.html;
                try_files $uri $uri/ /index.html;
        }

}
```
然后在项目中使用 **a** 标签或者其他的跳转方式跳转，服务器会请求到转发的 **html** 以及 **js**、**css** 等静态资源。
注意： 要配置 **webpack** 构建文件中的 **publicPath** 公共资源请求路径，按照相对于的路径请求相对应的资源 **（js, css……）**
            </div>
