<h2 id="删除docker组件">删除docker组件</h2>
<p><strong>查看是否已经安装的Docker软件包</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> yum list installed <span class="token operator">|</span> <span class="token function">grep</span> docker
</code></pre>
<p><strong>卸载列出的所有包</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> yum -y remove  \
docker docker-common docker-selinux  \
docker-engine docker-engine-selinux container-selinux docker-ce
</code></pre>
<p><strong>删除所有的镜像、容器、数据卷、配置文件等</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">rm</span> -rf /var/lib/docker
</code></pre>
<h2 id="安装docker组件">安装docker组件</h2>
<h3 id="安装docker">安装docker</h3>
<p><strong>1.1 安装docker(官方源方式)</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment">#安装curl</span>
yum <span class="token function">install</span> -y curl

<span class="token comment">#从官方源安装docker</span>
curl -fsSL https://get.docker.com <span class="token operator">|</span> <span class="token function">bash</span> -s docker --mirror Aliyun

<span class="token comment">#添加进启动菜单</span>
<span class="token function">sudo</span> systemctl start docker
<span class="token function">sudo</span> systemctl <span class="token function">enable</span> docker
</code></pre>
<p><strong>1.2 安装docker(yum源方式)</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment">#安装源</span>
<span class="token function">sudo</span> yum-config-manager --add-repo  \
https://download.docker.com/linux/centos/docker-ce.repo

<span class="token comment">#安装依赖</span>
<span class="token function">sudo</span> yum <span class="token function">install</span> -y yum-utils device-mapper-persistent-data lvm2

<span class="token comment">#安装社区stable版docker</span>
yum <span class="token function">install</span> docker-ce 

<span class="token comment">#添加进启动菜单</span>
<span class="token function">sudo</span> systemctl start docker
<span class="token function">sudo</span> systemctl <span class="token function">enable</span> docker
</code></pre>
<h3 id="安装web管理">2.安装WEB管理</h3>
<p>web管理使用Portainer</p>
<blockquote>
<p><a href="https://portainer.readthedocs.io/en/stable/deployment.html#quick-start">https://portainer.readthedocs.io/en/stable/deployment.html#quick-start</a></p>
</blockquote>
<p><strong>2.1 简单安装</strong></p>
<pre class=" language-bash"><code class="prism  language-bash">docker volume create portainer_data
docker run -d -p 9000:9000 --name portainer  \
--restart always -v /var/run/docker.sock:/var/run/docker.sock  \
-v portainer_data:/data portainer/portainer
</code></pre>
<p><strong>2.2 生产环境安装(TLS)</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment">#生成TLS文件</span>
openssl genrsa -out portainer.key 2048
openssl ecparam -genkey -name secp384r1 -out portainer.key
openssl req -new -x509 -sha256 -key portainer.key -out portainer.crt -days 3650

<span class="token comment">#安装TLS版</span>
docker run -d -p 443:9000 --name portainer  \
--restart always -v ~/local-certs:/certs  \
-v portainer_data:/data portainer/portainer  \
--ssl --sslcert /certs/portainer.crt  \
--sslkey /certs/portainer.key
</code></pre>
<h3 id="安装docker-管理api接口">3.安装Docker 管理API接口</h3>
<pre class=" language-bash"><code class="prism  language-bash">docker run -ti -d -p 2375:2375 --hostname<span class="token operator">=</span><span class="token variable">$HOSTNAME</span>  \
--name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock  \
-e PORT<span class="token operator">=</span>2375 shipyard/docker-proxy
</code></pre>
<h3 id="修改docker-配置文件">4.修改Docker 配置文件</h3>
<p><strong>4.1 直接修改配置文件</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> vim /usr/lib/systemd/system/docker.service
</code></pre>
<p>相关操作：</p>
<blockquote>
<p>ExecStart=/usr/bin/dockerd --insecure-registry 192.168.0.153:5000</p>
</blockquote>
<p><strong>4.2 使用json方式配置</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment">#建立配置文件目录</span>
<span class="token function">sudo</span> <span class="token function">mkdir</span> -p /etc/docker

<span class="token comment">#注册加速器及私有库地址</span>
<span class="token function">sudo</span> <span class="token function">tee</span> /etc/docker/daemon.json <span class="token operator">&lt;&lt;</span>-<span class="token string">'EOF'</span> 
<span class="token punctuation">{</span> 
  <span class="token string">"registry-mirrors"</span><span class="token keyword">:</span> <span class="token punctuation">[</span><span class="token string">"https://wv49dcaq.mirror.aliyuncs.com"</span><span class="token punctuation">]</span>,
  <span class="token string">"insecure-registries"</span><span class="token keyword">:</span> <span class="token punctuation">[</span>
    <span class="token string">"172.16.60.65:5000"</span>,
    <span class="token string">"172.16.65.166:5000"</span>,
    <span class="token string">"registry.cn-hangzhou.aliyuncs.com"</span>
<span class="token punctuation">]</span>
<span class="token punctuation">}</span>
EOF
</code></pre>
<h3 id="保存及重启docker">5.保存及重启Docker</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment">#重启Docker服务</span>
<span class="token function">sudo</span> systemctl daemon-reload
<span class="token function">sudo</span> systemctl restart docker
</code></pre>
<h2 id="docker的使用">Docker的使用</h2>
<h3 id="外网环境需登录私有公网库">1.外网环境需登录私有公网库</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment">#命令</span>
<span class="token function">sudo</span> docker login --username<span class="token operator">=</span><span class="token variable">$UserName</span>  \
registry.cn-hangzhou.aliyuncs.com

<span class="token comment">#例子</span>
<span class="token function">sudo</span> docker login --username<span class="token operator">=</span>yangbo@ybdev  \
registry.cn-hangzhou.aliyuncs.com
</code></pre>
<h3 id="相关build容器命令">2.相关build容器命令</h3>
<p>自动化脚本</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token shebang important">#!/bin/bash</span>
<span class="token comment"># author:yangbo</span>
<span class="token comment"># email:yangbo@yungengxin.com</span>
<span class="token comment"># 实现制作过程中自动化的build/tag/push</span>
<span class="token comment"># 使用前需登录公网账号</span>

M_REGISTRY1<span class="token operator">=</span><span class="token string">"172.16.60.65:5000"</span>
M_REGISTRY2<span class="token operator">=</span><span class="token string">"172.16.65.166:5000"</span>
M_REGISTRY3<span class="token operator">=</span><span class="token string">"registry.cn-hangzhou.aliyuncs.com/yjf/"</span>

M_WORKPATH<span class="token operator">=</span><span class="token string">"/root/projects/build/"</span>

M_ARGNUM<span class="token operator">=</span>$<span class="token comment">#</span>
M_OPTION<span class="token operator">=</span><span class="token variable">$1</span>
M_ARG1<span class="token operator">=</span><span class="token variable">$2</span>
M_ARG2<span class="token operator">=</span><span class="token variable">$3</span>
M_ARG3<span class="token operator">=</span><span class="token variable">$4</span>

helpfunction<span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token keyword">echo</span> <span class="token string">"<span class="token variable">$0</span> OPTION ARG..."</span>
	<span class="token keyword">echo</span> <span class="token string">"OPTION:"</span>
	<span class="token keyword">echo</span> <span class="token string">"	help		show help message."</span>
	<span class="token keyword">echo</span> <span class="token string">"	build		build a docker image"</span>
	<span class="token keyword">echo</span> <span class="token string">"		-arg1		image name"</span>
	<span class="token keyword">echo</span> <span class="token string">"		-arg2		version"</span>
	<span class="token keyword">echo</span> <span class="token string">"		-agr3		source path"</span>
	<span class="token keyword">echo</span> <span class="token string">"	fbuild		faster build a docker image"</span>
	<span class="token keyword">echo</span> <span class="token string">"		-arg1		image show name"</span>
<span class="token punctuation">}</span>

checkfunction<span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token keyword">echo</span> <span class="token string">"check..."</span>
	<span class="token keyword">if</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_ARGNUM}</span> <span class="token operator">!=</span> 4 <span class="token punctuation">]</span><span class="token punctuation">]</span>
	<span class="token keyword">then</span>
		<span class="token keyword">return</span> <span class="token variable"><span class="token variable">$((</span><span class="token number">1</span><span class="token variable">))</span></span>
	<span class="token keyword">elif</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_OPTION}</span> <span class="token operator">=</span> <span class="token string">""</span> <span class="token punctuation">]</span><span class="token punctuation">]</span>
	<span class="token keyword">then</span>
		<span class="token keyword">return</span> <span class="token variable"><span class="token variable">$((</span><span class="token number">1</span><span class="token variable">))</span></span>
	<span class="token keyword">elif</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_ARG1}</span> <span class="token operator">=</span> <span class="token string">""</span> <span class="token punctuation">]</span><span class="token punctuation">]</span>
	<span class="token keyword">then</span>
		<span class="token keyword">return</span> <span class="token variable"><span class="token variable">$((</span><span class="token number">1</span><span class="token variable">))</span></span>
	<span class="token keyword">elif</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_ARG1}</span> <span class="token operator">=</span> <span class="token string">""</span> <span class="token punctuation">]</span><span class="token punctuation">]</span>
	<span class="token keyword">then</span>
		<span class="token keyword">return</span> <span class="token variable"><span class="token variable">$((</span><span class="token number">1</span><span class="token variable">))</span></span>
	<span class="token keyword">elif</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_ARG1}</span> <span class="token operator">=</span> <span class="token string">""</span> <span class="token punctuation">]</span><span class="token punctuation">]</span>
	<span class="token keyword">then</span>
		<span class="token keyword">return</span> <span class="token variable"><span class="token variable">$((</span><span class="token number">1</span><span class="token variable">))</span></span>
	<span class="token keyword">fi</span>
	<span class="token keyword">return</span> <span class="token variable"><span class="token variable">$((</span><span class="token number">0</span><span class="token variable">))</span></span>
<span class="token punctuation">}</span>

create_version_function<span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token function">mkdir</span> -p <span class="token variable">${M_WORKPATH}</span>
	<span class="token function">mkdir</span> -p <span class="token variable">${M_WORKPATH}</span>/<span class="token variable">${M_ARG1}</span>
	version_path<span class="token operator">=</span><span class="token variable">${M_WORKPATH}</span>/<span class="token variable">${M_ARG1}</span>/version
	version<span class="token operator">=</span>0
	<span class="token keyword">if</span> <span class="token function">test</span> -s <span class="token variable">${version_path}</span>
	<span class="token keyword">then</span>
	<span class="token comment">#	echo "${M_WORKPATH}/${M_ARG1} version exist!"</span>
		version<span class="token operator">=</span><span class="token variable"><span class="token variable">$(</span><span class="token function">cat</span> $<span class="token punctuation">{</span>version_path<span class="token punctuation">}</span><span class="token variable">)</span></span>
		version<span class="token operator">=</span><span class="token variable"><span class="token variable">`</span><span class="token function">expr</span> $<span class="token punctuation">{</span>version<span class="token punctuation">}</span> + 1<span class="token variable">`</span></span>
	<span class="token keyword">else</span>
		version<span class="token operator">=</span>1
	<span class="token keyword">fi</span>
	<span class="token keyword">echo</span> <span class="token variable">${version}</span> <span class="token operator">&gt;</span> <span class="token variable">${version_path}</span>
	<span class="token keyword">echo</span> <span class="token string">"NOW VERSION : <span class="token variable">${version}</span>"</span>
<span class="token punctuation">}</span>

buildfunction<span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token keyword">echo</span> <span class="token string">"build image name:<span class="token variable">${M_ARG1}</span>"</span>
	docker build -t <span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span> <span class="token variable">${M_ARG3}</span>
	docker tag <span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span> <span class="token variable">${M_ARG1}</span>:latest
	docker tag <span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span> <span class="token variable">${M_REGISTRY1}</span>/<span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span>
	docker tag <span class="token variable">${M_ARG1}</span>:latest    <span class="token variable">${M_REGISTRY1}</span>/<span class="token variable">${M_ARG1}</span>:latest
	docker tag <span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span> <span class="token variable">${M_REGISTRY2}</span>/<span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span>
	docker tag <span class="token variable">${M_ARG1}</span>:latest    <span class="token variable">${M_REGISTRY2}</span>/<span class="token variable">${M_ARG1}</span>:latest
	docker tag <span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span> <span class="token variable">${M_REGISTRY3}</span>/<span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span>
	docker tag <span class="token variable">${M_ARG1}</span>:latest    <span class="token variable">${M_REGISTRY3}</span>/<span class="token variable">${M_ARG1}</span>:latest
	docker push <span class="token variable">${M_REGISTRY1}</span>/<span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span>
	docker push <span class="token variable">${M_REGISTRY1}</span>/<span class="token variable">${M_ARG1}</span>:latest
	docker push <span class="token variable">${M_REGISTRY2}</span>/<span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span>
	docker push <span class="token variable">${M_REGISTRY2}</span>/<span class="token variable">${M_ARG1}</span>:latest
	docker push <span class="token variable">${M_REGISTRY3}</span>/<span class="token variable">${M_ARG1}</span><span class="token keyword">:</span><span class="token variable">${M_ARG2}</span>
	docker push <span class="token variable">${M_REGISTRY3}</span>/<span class="token variable">${M_ARG1}</span>:latest
<span class="token punctuation">}</span>

fastbuildfunction<span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token function">mkdir</span> -p <span class="token variable">${M_WORKPATH}</span>
	<span class="token function">mkdir</span> -p <span class="token variable">${M_WORKPATH}</span>/<span class="token variable">${M_ARG1}</span>
	M_ARG3<span class="token operator">=</span><span class="token variable">${M_WORKPATH}</span>/<span class="token variable">${M_ARG1}</span>
	M_ARG1<span class="token operator">=</span><span class="token variable">${M_ARG1}</span>_image
	M_ARG2<span class="token operator">=</span>0.1
	buildfunction
<span class="token punctuation">}</span>

<span class="token keyword">echo</span> <span class="token string">"AUTO BUILD DOCKER SHELL"</span>
<span class="token keyword">echo</span> <span class="token string">"arg num:<span class="token variable">${M_ARGNUM}</span>"</span>

<span class="token keyword">if</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_OPTION}</span> <span class="token operator">=</span> <span class="token string">"help"</span> <span class="token punctuation">]</span><span class="token punctuation">]</span>
<span class="token keyword">then</span>
	helpfunction
<span class="token keyword">elif</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_OPTION}</span> <span class="token operator">=</span> <span class="token string">"build"</span> <span class="token punctuation">]</span><span class="token punctuation">]</span>
<span class="token keyword">then</span>
	checkfunction
	<span class="token keyword">if</span> <span class="token punctuation">[</span> <span class="token variable">$?</span> <span class="token operator">==</span> 0 <span class="token punctuation">]</span>
	<span class="token keyword">then</span>
		buildfunction
	<span class="token keyword">else</span>
		helpfunction
	<span class="token keyword">fi</span>
<span class="token keyword">elif</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_OPTION}</span> <span class="token operator">=</span> <span class="token string">"fbuild"</span> <span class="token punctuation">]</span><span class="token punctuation">]</span>
<span class="token keyword">then</span>
	fastbuildfunction
<span class="token keyword">elif</span> <span class="token punctuation">[</span><span class="token punctuation">[</span> <span class="token variable">${M_OPTION}</span> <span class="token operator">=</span> <span class="token string">"fv"</span> <span class="token punctuation">]</span><span class="token punctuation">]</span>
<span class="token keyword">then</span>
	create_version_function
	fastbuildfunction
<span class="token keyword">else</span>
	helpfunction
<span class="token keyword">fi</span>
	
</code></pre>
<p>脚本使用方式:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># 普通方式制作镜像</span>
./autodocker.sh build binname_image version /path/to 

<span class="token comment"># 快速模式制作镜像</span>
./autodocker.sh fbuild binname 

<span class="token comment"># 一键制作镜像(含版本号处理)</span>
./autodocker.sh fv binname 
</code></pre>
<h3 id="创建registoy库">2.创建Registoy库</h3>
<pre class=" language-bash"><code class="prism  language-bash">docker run -d -p 5000:5000  \
--restart always --name registry registry:2
</code></pre>

