# YZMOHI-Deployment-Doc

## Step1: Init DigitalOcean Droplets(初始化DigitalOcean Droplets部署环境)


* ###Root Login(Root权限登录)
    <br />
    `ssh root@SERVER_IP_ADDRESS`  
    <br />
* ###Create a New User(新建用户)
 <br />
    `adduser depolyer` 
 <br />
 
* ###Root Privileges(赋予root权限)  
  <br />
  * 赋予root权限
  <br />    
  	`gpasswd -a _deployer_ sudo`
  <br />	    
  
  * 切换到用户 deployer
  <br />    
  	`su - deployer`
  <br />	
  * 退出当前用户
  <br />    
  	`exit`
  <br />	
  
##Step2: Install Ruby on Rails on Ubuntu 14.04 using RVM(通过RVM安装Ubuntu 14.04的Ruby on Rail环境)
<br />

*  升级系统环境，安装Ruby附件
<br /> 
   				
		apt-get update
		apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties
<br />    

*  通过RVM安装Ruby on Rails环境
    <br />     
    *  下列命令通过rvm快速安装Ruby on Rails环境:
    <br />     
    `\curl -sSL https://get.rvm.io | bash -s stable --rails`
    <br />    
   （注：首次安装会提示添加gpg key，按照提示操作即可）
    <br />     
    安装完成后执行，运行rvm（按提示操作）
    <br />    
    `source /home/deployer/.rvm/scripts/rvm`
    <br /> 
    <br />
*  配置Git
    <br />    
    
    * 将SSH key加入git，否则vps无法访问git
    <br />    
    
    * 生成SSH key添加到先关git账号，怎样添加详见
    <br />    
    [Using SSH agent forwarding](https://developer.github.com/guides/using-ssh-agent-forwarding/#setting-up-ssh-agent-forwarding)
    <br />    
    
    * 验证是否能够登录git
    <br />	
   `ssh -T git@bitucket.org`
    <br />		
    
    * 安装Node.js
    <br />
         
        Since Rails ships with so many dependencies these days, we're going to need 
        to install a Javascript runtime like NodeJS.This lets you use Coffeescript 
        and the Asset Pipeline in Rails which combines and minifies your javascript 
        to provide a faster production environment.
    
    APT安装
    
		sudo apt-get update  
		sudo apt-get install nodejs
    
	If the package in the repositories suits your needs, this is all that you 
	need to do to get set up with Node.js. In most cases, you'll also want to also 	install npm, which is the Node.js package manager. You can do this by typing:

    安装版本控制:
    <br />	
    	This will allow you to easily install modules and packages to use with Node.js.
     <br />		
    		`sudo apt-get install npm`  
    <br />
    	（ Because of a conflict with another package, the executable from the 
    	Ubuntu repositories is callednodejs instead of node. Keep this in mind as 
    	you are running software. ） 
    <br />	
* 安装配置MySQL
<br />	
	You can install MySQL server and client from the packages in the Ubuntu repository. 	As part of the installation process, you'll set the password for the root user. This 	information will go into your Rails app's database.yml file in the future.
<br />	 
	`sudo apt-get install mysql-server mysql-client libmysqlclient-dev` 
<br />     
	(Installing the libmysqlclient-dev gives you the necessary files to compile the 	mysql2 gem which is what Rails will use to connect to MySQL when you setup your Rails 	app.)

##Step3: Deploy a Rails 4 App With Git and Capistrano（通过Git和Capistrano将Rials4应用部署到VPS上）
*	本地环境
<br />	

	*	设置APP各项参数
	<br />	
	
	*	完成应用程序
	<br />	
   特别注意配置 database.yml
   <br />	
   		Edit your database.yml file to configure it for your development and 
   		production database, then try it out locally at http://localhost:3000 
   		to make sure it's working properly (test production after deployment).

	*	git配置
		*	配置本地git
		
				git config --global user.name "Your name"
       			git config --global user.email "your_email@example.com"
       			cd /your/project/folder
				git init
				touch .gitignore	
       	
		*	配置 .gitignore
		<br />	
			This file contains a list of files (and patterns of files) to ignore 
			and not include in your repository. This is useful so you don't 
			include stupid stuff like .DS_Store but also to keep sensitive files 
			like database.yml out of it so you're not storing passwords where
			everybody (well, those who have access to your repository) can see them.
			You can include other stuff as you see fit (e.g., I like to exclude 
			any sqlite databases I might be using as well as static asset files that
		    I've generated for testing but which aren't part of my app) but you at 
		    least want to add these: database config, Mac junk, log files, and
		    temp files.
		    
				/config/database.yml
				.DS_Store
				/log/*
				/tmp/*
				
*	服务器端配置
    <br />	
	添加deployer用户和用户组(已经在 初始化DigitalOcean Droplets部署环境 中完成)
	<br />	
*	Capify(配置Capistrano)

	*	[Capistrano Handbook] 阅读配置手册

	*	安装Capistrano（在本地环境）
	<br />	
		1. adding it to your app's Gemfile and runningbundle install. You don't 
		need it on the production server, so you can just add it under the 
		"development" group in your Gemfile. 
	<br />
    	>Gemfile.rb
   					
				group :development do
					gem 'capistrano'
   				end
		
		2. And install with bundler (from your app's root).
		<br />	
        		`bundle install`
        <br />	
        
		3. Once installed, you'll need to "capify" your project. This will generate a 
		few files (namely /config/deploy.rb and /Capfile). The deploy.rb is where you
		 can add or write custom scripts to help automate your deployment tasks and 
		 save your carpal tunnel for more important application programming.
		 <br />		
		 		`cd /path/to/your/project capify .`
		 <br />		
	*	Capistrano settings（配置Capistrano）
	<br />	
		1. Once your project is capified, open up your editor of choice and 
		modify/config/deploy.rb (here's the [final example]). There's a bunch 
		of stuff in there, but first I'm going to focus on the top-most settings, 
		denoted with various setcommands.
		<br />	
		我的配置文件如下
		<br />	
		> /config/deploy.rb 
		<br />	
			
				set :application, 'AdiantumBlog'
 				set :repo_url, 'git@bitbucket.org:allenzhong/adiantumblog.git'
 				set :deploy_to, '~/YueZhongMOHI'
 				set :scm, :git
 				set :branch, "master"
 				set :user, "deployer"
 				set :use_sudo, false
 				set :rails_env, "production"
 				set :deploy_via, :copy
 				set :keep_releases, 5
 				server "198.199.114.11", user: "deployer", roles: [:app, :web, :db], :primary =>true
 				

			配置说明:
			<br />	
			a	. First, you need to tell Capistrano your app's name and the clone path 			to your app's remote repository.

				set :application, "YourApplicationName"
				set :repository, "git@github.com:your-username/your-repository-name.git"

			b	. Next, tell it where your app needs to be installed on your production 			server (this is the path to where you want your app to live, starting from 			your server's root). The path will be the location you pointed your server to 			that I covered in my previous post. You can leave this out if you want to 			deploy to Capistrano's default location of /u/apps/#{{application} }.
			<br />	
				`set :deploy_to, "/path/to/your/app"`
			<br />	
			

			c	. Set the type of repository you're using (e.g., GIT, SVN, etc.) and 			which branch you want to deploy from (usually "master").
				
				set :scm, :git
				set :branch, "master"

			d	. Set the user you want to use for your server's deploys.
			<br />	
				`set :user, "deployer"`
			<br />	
			e	. I change another setting called use_sudo 
			to false so commands are executed with the user's permissions unless I 			specify otherwise. If you added your user to the "deployers" group (which has 			write-access to your app's directory tree) you probably won't need to use 			sudo much, if at all.
			<br />	
				`set :use_sudo, false`
			<br />	
			f	. Set your rails environment (this is deploying to the production server, 			so I'm setting it to "production", but you might want to deploy to a staging 			server with a custom environment). You can re-use this variable in your 			scripts whenever you need to specify the environment so you only need to 			change it in one place to keep things DRY.
			<br />	
				`set :rails_env, "production"`
			<br />	
			g	. Tell Capistrano how you'd like to make updates. There are many 			different ways of doing it, but for simplicity's sake, I'm going to stick 			with the most straight-forward method, namely copy. This will clone your 			entire repository (download it from the remote to your local machine) and 			then upload the entire app to your server.
			<br />	
				`set :deploy_via, :copy`
			<br />	
			h	. You can also specify how many releases Capistrano should store on your 			server's harddrive. This is handy in case you ever want to rollback to a 			previous version quickly in case your newly deployed code blew something up 			and you need to put out the fires. But you probably want this to be a finite 			number so you don't fill up your disk with inactive versions of your app. I 			keep five releases.
			<br />	
				`set :keep_releases, 5`
			<br />	
			i	. The last setting you need to handle is where on the internet Capistrano 			can find your server. This could be your domain name or the IP address. I'm 			assuming this deployment is for a smallish MVP-type app where everything is 			on the same machine (database, app, server, etc.). In this case you can use 			Capistrano's server setting.
			<br />	
				`server "198.199.114.11", user: "deployer", roles: 				[:app, :web, :db], :primary => true`
			<br />	
			j	. 配置 /config/deploy/production.rb 文件
			<br />	
			If you want to do fancier deployments by splitting things up for scaling 			(like separating your database from your application server), you'll want to 			use Capistrano's "roles" to point it to the different places where things are 			installed. Use multiple roles instead of the server command (it accomplishes 			the same thing with greater granularity).

			>	/config/deploy/production.rb
			
			
				role :app, %w{deployer@198.199.114.11}
		 		role :web, %w{deployer@198.199.114.11}
				role :db,  %w{deployer@198.199.114.11}
				server '198.199.114.11', user: 'deployer', roles: %w{web app}, 				my_property: :my_value
				

			k	.配置Capfile
			<br />	
			在Capfile中加入
			<br />	
				`Rake::Task[:production].invoke`
		    <br />	
			否则部署时会报错:
			<br />	
				**Stage not set, please call something such as cap production deploy, 				where production is a stage you have defined.**
			<br />	
			l	. Capistrano部署前要将以下配置加入 /config/environments/production.rb 用于assets 			precompile

				config.assets.compile = true
				config.assets.precompile =  ['*.js', '*.scss', '*.scss.erb']
				config.serve_static_assets = false


*	链接部署到VPS服务器

	1.	链接部署
	<br />	
	在本地Terminal中App的目录下执行,就可以将本地app部署到VPS上的相应文件夹中(最后第3步执行)
	<br />	
		`cap production deploy`
	<br />	
	
	2.	在服务器上将会创建以下文件夹
	<br />	
	This should setup the following directories (if you've specified to install your app in 
	/var/www/example.com/):
	<br />	
	
			/var/www/example.com/current
			/var/www/example.com/shared
			/var/www/example.com/releases
			

		三个文件夹的作用：
		<br />	
		The releases directory is where copies of all your actual code are stored.shared 		is a place where you can put common, shared files like logs, static assets, and 		in our case, config files like database.yml. current is simply a symbolic link 		that points to the current release inside the releases directory (Capistrano 		updates this for you on each other deploy, so don't worry about it).
		<br />	
	3.	检查部署状态
	<br />	
	Run the deploy:checkcommand. This will check your local environment and your server 	and try to locate any possible issues. If you see any errors, fix them first, and 	then run the command again until you don't have any more errors.
	<br />	
		`cap deploy:check`
	<br />	
	Any number of things could go wrong here, but the most likely issues will be some sort of 	authentication or file permission issues with your server and the user you're logging in 	with SSH (I've lost count of the number of times I've wanted to punch my computer due to 	some unknown error only to discover I had set the file permissions incorrectly).
	<br /><br />
*	Database.yml文件
<br />	
	I excluded Database.yml from your repository by adding it to your.gitignore file. This is 	good (don't store sensitive data like passwords in your repository). However, when you run 	Capistrano to update your server, it only copies the files that are in your repository (and 	Rails actually needs to use thatdatabase.yml file to hook up to your server's database, 	unless you don't plan on storing any data whatsoever). So, the problem is: we need to get 	that config file onto the server somehow, securely, and make sure Rails knows where to find 	it.
	<br />	
  	For this, I just manually put the file on my server in a shared location that only I (and 	the app) have read-access to.

	1.	First, make a new config directory in the shared directory that Capistrano created.
				
			ssh deployer@xxx.xxx.xxx.xxx
			mkdir /home/deployer/YueZhongMOHI/shared/config
	2.	Leave the server and back on your local machine, in your app's root directory, copy the 	database.yml file to the server (NOTE: unintuitively, you need to specify the port number 	with a capital P when using the scp command).

			scp ./config/database.yml  deployer@xxx.xxx.xxx.xxx:/home/deployer/YueZhongMOHI/			shared/config/database.yml
			
		**Note: the above command is one long line (in case it's wrapping in your browser**
	3. 	执行部署命令将remote git的Rails App部署到vps
	<br />	
		`cap production deploy`
		
	4. 	对于本app利用paperclip 以及 kindeditor上传图片，每次部署新版本的app之后原图片链接会失效，需要做	symlink防止新部署的文件覆盖上传文件夹

		paperclip 图片上传保存路径：
		<br />	
				`public/system`
		<br />	
		kindeditor 图片上传保存路径：
		<br />	
				`public/uploads`
		<br />	
		a	. 问题产生原因
		<br />	
		Problems might arise at deployment time if you are managing your Rails application with 		Capistrano. By default Capistrano doesn’t support an additional public/system folder 		unless it is under version control and each time you’ll deploy your Rails application the 		path stored for the uploaded files will be invalidated.
		<br />	
		b	. 解决图片消失问题
		<br />	
		（1）Restored symlink 
			 <br />		
				symlink for paperclip: 
	 		<br />		
				`ln -s /home/user/app/shared/public/system /home/user/app/current/public/system`
			<br />	
	 			symlink for kindeditor:
	 		<br />	
	 		
				`ln -s /root/YueZhongMOHI/shared/public/uploads /root/YueZhongMOHI/current/public/uploads`
			<br />	
		（2）Add **public/system public/uploads** folder to .gitignore on development server
		<br />	
		（3）Add in deploy.rb
		<br />	
			`set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system public/uploads}`


## Step4: Deploy a Rails App with Passenger and Nginx on Ubuntu 14.04（在Ubuntu 14.04系统上通过Passenger and Nginx部署和启动Rails应用）

*	部署前Rails App assets 预编译准备工作

	1. Capistrano部署前要将以下配置加入本地文件夹 /config/environments/production.rb 用于assets 	precompile

			config.assets.compile = true
			config.assets.precompile =  ['*.js', '*.scss', '*.scss.erb']
			config.serve_static_assets = false

	2. 进入远端服务current文件夹
	<br />	
		a	. 执行以下命令，安装app所需gem
		<br />	
			`bundle install`
		<br />	
		b	. 创建mysql数据库（对应database.yml所配置的数据库名称）
		<br />	
			启动MySQL服务:
			<br />	
				`sudo service mysql start`
			<br />	
			登录 MySQL 服务器:
			<br />	
			以 **root** 用户身份登录到 MySQL ，在系统提示你输入密码时直接输入你在安装 MySQL 时创建的 			root 密码即可。（注：这里的密码可以和你 Linux 系统的 root 密码不同）
			<br />	
				`mysql -u root -p`
			<br />	
			创建数据库:
			<br />	
				``mysql> CREATE DATABASE `blog`;``
			<br />	
			授予访问权限:
			<br />	
				``mysql> GRANT ALL PRIVILEGES ON `blog`.* TO 'deployer'@'localhost';``
			<br />	
			刷新 MySQL 权限以接受你之前做出的所有更改:
			<br />	
				`mysql> FLUSH PRIVILEGES;`
			<br />	
			退出 mysql 客户端:
			<br />	
				`mysql> exit`
			<br />	
		c	. 执行migrate，seed，创建数据库表，写入默认值
		<br />	
			执行migration:
			<br />	
				`bin/rake db:migrate RAILS_ENV=production`
			<br />	
			执行seeds写入预配置数据:
			<br />	
				`bin/rake db:seed RAILS_ENV=production`
			<br />	
		d	.(optional)对于影像馆app
		<br />	
		（1）用到rails kindeditor
		<br />	
	 	Rails4 in production mode
	 	<br />	
	  	In Rails 4.0, precompiling assets no longer automatically copies non-JS/CSS 		assets from vendor/assets and lib/assets. see https://github.com/rails/rails/		pull/7968 In Rails 4.0's production mode, please run 'rake kindeditor:assets', 		this method just copy kindeditor into public folder.
	  	<br />	
		执行以保证在生产环境中加载kind editor:
		<br />	
			`rake kindeditor:assets`
		<br />	
		（2）安装ImageMagick
		<br />	
			`sudo apt-get install imagemagick`
		<br />	
		e	.预编译assets
		<br />	
			`RAILS_ENV=production rake assets:precompile`
		<br />	
	 	f	. 添加生产环境所需的secrect_key_base,否则在生产环境启动会报错
	 	<br />	
		>config/secrets.yml
		<br />	
				
			#Do not keep production secrets in the repository,
			#instead read values from the environment.
			production: 
				secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
									
	    (1) In the terminal of our production server you will execute the next command:
		<br />	
			`RAILS_ENV=production rake secret`
		<br />	
			This will give a large string with letters and numbers, this is what you 			need, so copy that (we will refer to that code as GENERATED_CODE).
		<br />	
		(2.1) Now if we login as root user to our server we will need to find this file 		and open it: 
		<br />	
			`vim /etc/profile`
		<br />	
			Then we go to the bottom of the file ("SHIFT + G" for capital G in VI)
			And we will write our environment variable with our GENERATED_CODE (Press "i" 			key to write in VI), be sure to be in a new line at the end of the file:
			<br />	
				`export SECRET_KEY_BASE=GENERATED_CODE`
			<br />	
				Having written the code we save the changes and close the file (we push 				"ESC" key and then write ":x" and "ENTER" key for save and exit in VI)
			<br />	
	 	(2.2) But if we login as normal user, lets call it example_user for this gist, we 		will need to find one of this other files:
	 	<br />
							
			vi ~/.bash_profile
			vi ~/.bash_login
			vi ~/.profile
			
		These files are in order of importance, that means that if you have the first 		file, then you wouldn't need to write in the others. So if you found this 2 files 		in your directory "~/.bash_profile" and "~/.profile" you only will have to write 		in the first one "~/.bash_profile", because linux will read only this one and the 		other will be ignored.
		<br />	
		Then we go to the bottom of the file ("SHIFT + G" for capital G in VI) 
		<br />	
		And we will write our environment variable with our GENERATED_CODE (Press "i" key 		to write in VI), be sure to be in a new line at the end of the file:
		<br />	
			`export SECRET_KEY_BASE=GENERATED_CODE`
		<br />	
		Having written the code we save the changes and close the file (we push "ESC" key 		and then write ":x" and "ENTER" key for save and exit in VI)
		<br />	
		(3) We can verify that our environment variable is properly set in linux with 		this command:
		<br />	
			`printenv | grep SECRET_KEY_BASE`
		<br />	
		or with:
		<br />	
			`echo $SECRET_KEY_BASE`
		<br />	
		When you execute this command, if everything went ok, it will show you the 		GENERATED_CODE that we generated before. Finally with all the configuration done 		you can deploy without problems your Rails app with Unicorn or other.
		<br />	
		Now when you close your shell terminal and login again to the production server 		you will have this environment variable set and ready to use it.
		<br />	
*	Install Passenger and Nginx(安装Passenger 和 Nginx)
<br />	
	使用APT安装:
	<br />	
	now install Passenger on Ubuntu with the Advanced Packaging Tool (APT), which is what 	we'll be using. In this manner, the installation and — even more importantly — the 	update process for Passenger with Nginx, is really simple.
	<br />	
		a	. install a PGP key:
		<br />	
			`sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 561F9B9CAC40B2F7`
		<br />	
		b	. Create an APT source file (you will need sudo privileges):
		<br />	
			`sudo vim /etc/apt/sources.list.d/passenger.list`
		<br />	
		And insert the following line in the file:
		<br />	
			`deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main`
		<br />	
		c	. Change the owner and permissions for this file
		<br />	    		    
			`sudo chown root: /etc/apt/sources.list.d/passenger.list`
			<br />	
			`sudo chmod 600 /etc/apt/sources.list.d/passenger.list`
		<br />	
		d	.Update the APT cache:
		<br />	
			`sudo apt-get update`
		<br />	
		e	. Finally, install Passenger with Nginx:
		<br />	
			`sudo apt-get install nginx-extras passenger`
		<br />	
			**安装过程会将Ruby版本改写成旧版本，配置nginx.conf时注意**

*	Set Up The Web Server(配置web服务)
<br />	
	1	. Open the Nginx configuration file:
	<br />	
		`sudo vim /etc/nginx/nginx.conf`
	<br />	
	2	. Uncomment both of them. Update the path in the line. They should look like 	this:
			
		passenger_root /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini;
		passenger_ruby /usr/local/bin/ruby;
		
	**用passenger-config --ruby-command找到正确的 passenger_ruby 路径**
	<br />	
	3	. user设置
	<br />
	(1) **修改成你的系统帐号名，不然项目目录 /home/jason/www 这里没有权限**
	>	/etc/nginx/nginx.conf
	<br />	
		`user deployer;`
	<br />	
	(2) Put the following into your nginx.conf in the http block:
	<br />	
	**防止出出现nogroup nobody错误**
	
		passenger_default_user custom_username;
		passenger_default_group custom_group;

	4	. We need to disable the default Nginx configuration. Open the Nginx config file:
	<br />	
		`sudo vim /etc/nginx/sites-available/default`
	<br />	
		Find the lines and Comment them out like this:
	<br />	
	
		#listen 80 default_server;
		#listen [::]:80 default_server ipv6only=on;

	5	. Now, create an Nginx configuration file for our app:
	<br />		
		`sudo vim /etc/nginx/sites-available/AdiantumBlog`
	<br />	
		a	. Add the following server block. The settings are explained below
		<br />	
		
		server { 
			listen 80 default_server;
			server_name 198.199.114.11;
			passenger_enabled on;
			# passenger_app_env development;
			root /home/deployer/YueZhongMOHI/current/public;
		}

	**注意，在初次部署时不要注释 passenger_app_env development; 这样可以看到passenger的错误页面一	遍差错，passenger默认启动模式是production，该环境下不显示错误页面，待错误解决后再注释掉用生产环境启	动**
	<br />	
		In this file, we enable listening on port 80, set your domain name, enable 		Passenger, 	and set the root to the public directory of our new project. The root 		line is the one 	you'll want to edit to match the upload location of your Rails 		app.
	<br />	
		b	. Create a symlink for it:
		<br />	
			`sudo ln -s /etc/nginx/sites-available/AdiantumBlog /etc/nginx/sites-enabled/AdiantumBlog`
		<br />	
		c	. Restart Nginx:
		<br />	
			`sudo nginx -s reload`
		<br />	
		d	. 注意调试完成后将
		<br />	
		**/etc/nginx/sites-available/AdiantumBlog 文件中的passenger_app_env注释掉，以生产环境		启动**
		
			server { 
			listen 80 default_server;
			server_name 198.199.114.11;
			passenger_enabled on;
			# passenger_app_env development;
			root /home/deployer/YueZhongMOHI/current/public;
			}
			
			
*	输入网站地址
<br />	
	出现任何问题通过以下命令查看nginx error.log查找报错
	<br />	
		`vim /var/log/nginx/error.log`
	<br />	
	This log file contains:
	<br />	
	1	.Phusion Passenger error messages.	
	<br />	 	2	.Everything that the application writes to STDERR. This typically consists of 	errors that the application encounters during startup, but not errors that it 	encounters when it’s handling requests.
	<br />	
	If you’re using Ruby on Rails, then you should also check out log/development.log and 	log/production.log. When an error occurs during request handling, it is typically 	logged here. This file does not contain errors that Rails encounters during startup.
	<br />	
	Finally, you should be aware that Phusion Passenger runs your application under the 	production environment by default, not development. You can change this using the 	rails_env option.











	

	
