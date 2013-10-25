Thinksns应用加入个人Profile的方法

###0.需求描述

海虾放出了4个新的应用,都支持个人主页(Profile)页面显示其应用的列表,对于这个功能感觉还是很需要的.所以本文的目的就是教大家如何将自己的应用的任意内容或列表加入到个人主页上,当别人只要浏览个人主页可以看到相应的应用内容.

应用下载地址:

[微博地址](http://demo.thinksns.com/t3/index.php?app=public&mod=Profile&act=feed&feed_id=57816&uid=2)

[blog.zip](http://demo.thinksns.com/t3/index.php?app=widget&mod=Upload&act=down&attach_id=31430)

[photo.zip](http://demo.thinksns.com/t3/index.php?app=widget&mod=Upload&act=down&attach_id=31433)

[group.zip](http://demo.thinksns.com/t3/index.php?app=widget&mod=Upload&act=down&attach_id=31434)

[vote.zip](http://demo.thinksns.com/t3/index.php?app=widget&mod=Upload&act=down&attach_id=31435)

###1.实现原理分析

1.  第一步需要定位哪些代码实现了个人主页的应用展示, 根据浏览器的标签分析, 找到个人主页下面应用的分类标签的实现源文件:

	模板文件:`_tab_menu.html`, 该文件主要是实现显示应用列表成为tab样式.
	
	文件路径:`/apps/public/Tpl/default/Profile/_tab_menu.html`
	
	源码分析: 通过源码发现,显示的应用使用`appArr`参数存储,渲染到模板,每个应用又调用了`{:U('public/Profile/appList',array('type'=>$key,'uid'=>$uid))}` 这个函数来生成应用tab下面的内容. 那么`appArr`是如何生成的呢,继续看下面.
	
	模板对应的Action函数:`public function _tab_menu ()`

	文件路径:`apps/public/Lib/Action/ProfileAction.class.php`
	
	源码分析: 该函数功能比较简单,就是查询app的数据表,遍历每个app,然后赋值给`appArr`,但是**并不是每个App都**都会加入,请看这行代码.
	
		if (method_exists($dao, 'profileContent')) {
			$appArr [$appName] = L ( 'PUBLIC_APPNAME_' . $appName );
		}
	
	说明需要存在一个`profileContent`方法的App才能够出现在这个列表里面!
	
	另外大家注意,他的应用名字并不是从应用设置里面获取的,而是从语言配置里面获取的,也就是说如果想更改个人主页上的应用的名字,需要更改语言配置(后台中有选项)对应的中文名. 例如
	
		`PUBLIC_APPNAME_BLOG  个人博客`
	
	附重要源码:


		<php> 
		      if(ACTION_NAME=='index'){ $tabClass['public'] = 'current'; }
		      elseif(ACTION_NAME=='data'){ $tabClass['public_data'] = 'current'; }
			  else{ $tabClass[$type] = 'current'; }
		</php>
		<div class="tab-menu clearfix">
		  <ul>
		    <li class="{$tabClass.public}"><span><a href="{:U('public/Profile/index',array('type'=>$type,'feed_type'=>'','uid'=>$uid))}">微博</a></span></li>
		    <li class="{$tabClass.public_data}"><span><a href="{:U('public/Profile/data',array('uid'=>$uid))}">资料</a></span></li>
			<volist name='appArr' id='vo'>
			    <li class="{$tabClass[$key]}"><span><a href="{:U('public/Profile/appList',array('type'=>$key,'uid'=>$uid))}">{$vo}</a></span></li>
			</volist>
		  </ul>
		</div>


		/**
		 * 个人主页标签导航
		 * @return void
		 */
		public function _tab_menu () {
			// 取全部APP信息
			$map['status'] = 1;
			$appList = model('App')->where($map)->field('app_name')->findAll();
			// 获取APP的HASH数组
			foreach ($appList as $app) {
				$appName = strtolower($app['app_name']);
				$className = ucfirst($appName);
				$dao = D($className.'Protocol', strtolower($className), false);
				if (method_exists($dao, 'profileContent')) {
					$appArr [$appName] = L ( 'PUBLIC_APPNAME_' . $appName );
				}
				unset ( $dao );
			}
			$this->assign ( 'appArr', $appArr );
			
			return $appArr;
		}


2.  第二步就是寻找函数`public/Profile/appList`,及其模板. 分析每个应用是如何实现显示的.

	模板文件: `appList.html`,实现显示应用内容
	
	文件路径: `/apps/public/Tpl/default/Profile/appList.html`
	
	源码分析: 内容很简单,只需要判断一下用户十分设置了隐私(不让别人产看),如果没有设置,那么就显示 `{$content}`, 也就是应用数据,那么该变量是如何生成的呢? 继续查看
	
	模板对应的Action函数: `public function appList ()`
	
	文件路劲: `apps/public/Lib/Action/ProfileAction.class.php`
	
	源码分析:	重点产看一下代码:

		$this->assign('type', $type);
		$className = ucfirst($type).'Protocol';
		$content = D($className, $type)->profileContent($this->uid);
		if (empty($content)) {
			$content = '暂无内容';
		}
	
	发现其实`$content`是通过调用相对应的应用的模型类中的`profileContent`方法获得. (D函数就是Thinkphp模型化函数,通过该函数可以获得相应的模型对象,也就可以使用相应的函数) 
	
	对于`classname`其实就获取了当前单击tab菜单后,访问的应用的名称!(使用GET获得).
	
	另外我们注意源码中`classname`中加入了`'Protocol'`,也就是说应用的模型类应该是有`Protocol`命名的.
	
	所以下面一节我们分析对应应用的模型类中的`profileContent`方法.
	
	附重要源码:
	
	
		<include file="__THEME__/public_header" />
		    <div id="page-wrap">
		        <div id="main-wrap">
		            <div class="profile-title  boxShadow">
						<include file="_top"/> 
					</div>
		            <div id="col" class="st-grid boxShadow minh content-bg">
		                <php>if($userPrivacy['space'] != 1){</php>
		                <include file="_sidebar"/>
		                    <div id="col5" class="st-index-main">
		                        <div class="extend-foot">
		                            <include file="_tab_menu"/>
		                            {$content}　　　　　　　　　　　　　
		                        </div>
		                    </div>
		                <php>}else{</php>
		                    <p class="extend">-_-。sorry！根据对方隐私设置，您无权查看TA的主页</p>
		                <php>}</php>
		            </div>
		            
		        </div>
		    </div>
		<include file="__THEME__/public_footer" />
		<script src="__THEME__/js/module.weibo.js"></script>


		/**
		 * 获取指定用户的应用数据列表
		 * @return array 指定用户的应用数据列表
		 */
		public function appList () {
			// 获取用户信息
			$user_info = model('User')->getUserInfo($this->uid);
			// 用户为空，则跳转用户不存在
			if (empty($user_info)) {
				$this->error(L('PUBLIC_USER_NOEXIST'));
			}
			// 个人空间头部
			$this->_top ();
			$this->_assignUserInfo($this->uid);
			$appArr = $this->_tab_menu();
			$type = t ( $_GET ['type'] );
			if (! isset ( $appArr [$type] )) {
				$this->error ( '参数出错！！' );
			}
			$this->assign('type', $type);
			$className = ucfirst($type).'Protocol';
			$content = D($className, $type)->profileContent($this->uid);
			if (empty($content)) {
				$content = '暂无内容';
			}
			$this->assign('content', $content);
			// 判断隐私设置
			$userPrivacy = $this->privacy($this->uid);
			if ($userPrivacy ['space'] !== 1) {
				$this->_sidebar ();
				// 档案类型
				$ProfileType = model ( 'UserProfile' )->getCategoryList ();
				$this->assign ( 'ProfileType', $ProfileType );
				// 个人资料
				$this->_assignUserProfile ( $this->uid );
				// 获取用户职业信息
				$userCategory = model ( 'UserCategory' )->getRelatedUserInfo ( $this->uid );
				if (! empty ( $userCategory )) {
					foreach ( $userCategory as $value ) {
						$user_category .= '<a href="#" class="link btn-cancel"><span>' . $value ['title'] . '</span></a>&nbsp;&nbsp;';
					}
				}
				$this->assign ( 'user_category', $user_category );
			} else {
				$this->_assignUserInfo ( $this->uid );
			}
			$this->assign ( 'userPrivacy', $userPrivacy );
			$this->setTitle ( $user_info ['uname'] . '的'.L ( 'PUBLIC_APPNAME_' . $type ) );
			$this->setKeywords ( $user_info ['uname'] . '的'.L ( 'PUBLIC_APPNAME_' . $type ) );
			$user_tag = model ( 'Tag' )->setAppName ( 'User' )->setAppTable ( 'user' )->getAppTags ( array (
					$this->uid
			) );
			$this->setDescription ( t ( $user_category . $user_info ['location'] . ',' . implode ( ',', $user_tag [$this->uid] ) . ',' . $user_info ['intro'] ) );
			$this->display ();
		}

3.  第三步就是寻找应用模型类中的`profileContent`方法,及其模板. 以Blog应用为模板!

	模型文件(Model): `BlogProtocolModel.class.php`
	
	文件路径: `/apps/blog/Lib/Model/BlogProtocolModel.class.php`
	
	源码分析: 该函数就是获取了该应用想要显示在个人主页上面的内容的模板,存于`$tpl`中. 对于Blog应用,其实就是首先获取日志分类,同时配置以下列表的参数,最后获取!
	
	模板文件: `profileContent.html`,该应用想要显示的内容模板.
	
	文件路径: `apps/blog/Tpl/default/Index/profileContent.html`
	
	源码分析: 内容很简单,只是包含了日志列表模板.`<include file="../Public/_bloglist" />`,实际上这里可以直接写你想要的样子, 只不过日志列表模板这里重复利用, 也就是说你在应用界面看到的日志列表, 再次利用到这里. 代码得到了复用. 对于日志列表模板, 这里就不再描述, 这就是应用本身的功能实现了.
	
	附重要源码:

		/**
		 * 在个人空间里查看该应用的内容列表
		 *
		 * @param integer $uid
		 *        	用户UID
		 * @return array 个人空间数据列表
		 */
		public function profileContent($uid) {
			$map ['uid'] = $uid;
			$map ['status'] = 1;
			$list = M ( 'blog' )->where ( $map )->order ( 'cTime DESC' )->findPage ( 20 );
			foreach ( $list ['data'] as $k => $v ) {
				if (empty ( $v ['category_title'] ) && ! empty ( $v ['category'] )) {
					$list ['data'] [$k] ['category_title'] = M ( 'blog_category' )->where ( 'id=' . $v ['category'] )->getField ( 'name' );
				}
			}
			$list ['titleshort'] = 200;
			$list ['suffix'] = '......';
			
			$tpl = APPS_PATH . '/blog/Tpl/default/Index/profileContent.html';
			return fetch ( $tpl, $list );
		}
		
		
		<link rel="stylesheet" href="__ROOT__/apps/blog/_static/blog.css"
			type="text/css" media="screen" charset="utf-8" />
		<link href="__PUBLIC__/js/pagination/pagination.css" rel="stylesheet"
			type="text/css" />
		<include file="../Public/_bloglist" />

4.  总结

	通过以上三步基本就分析了个人主页应用的显示原理! 下一章总结并列出将自己的应用加入到个人主页的步骤.
	
###2.操作步骤

*. 首先为你的应用的Model文件夹中加入 `*Appname*ProtocolModel.class.php`文件,其中Appname是你的应用名称. 或者复制一个从上面4个应用中,并改名字.

*. 实现函数`function profileContent`文件, 用于制造你需要在模板中显示的数据参数.

*. 在模板Tpl文件夹中加入`profileContent.html`文件, 这个文件你可以随意撰写,也就是说你可以在其中加入任何内容.

###3.实现效果

大家可以前往[@导游|idaoyoo](http://www.idaoyoo.com)查看个人主页,我自己加入了一个日程的应用,并且将日程的日历和日程列表都加入到了个人主页中. 具体效果请自行注册尝试, 当然如果是测试发的内容大家尽量删掉哦~~ Thanks very much!

###4.总结
