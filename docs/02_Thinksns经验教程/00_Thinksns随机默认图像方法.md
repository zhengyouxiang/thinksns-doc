Thinksns 随机默认头像的修改思路

###0. 需求描述

TS默认头像比较单调,而且考虑很多用户会比较懒,不自己添加头像,这样会让网站略显单调.于是考虑使用随机默认头像的方法,可以为偷懒的用户提供一个比较美观的头像,可以让微博界面瞬间有feel.

###1. 功能简介

实现的功能很简单,用户从注册开始,第一次出现头像的时候,会为改头像随机选择一张默认头像,并且永久分配给该用户,只要用户不自己上传,会一直使用该默认头像. 如果之前有用户注册了而没有上传头像,那么假如加入该功能,可以马上打开用户搜索界面,并且查看所有人的头像,这样就可以为所有用户的头像时分配一个随机头像,之后就永久使用.当然最后还是建站初就加入该功能.

###2. 实现原理
实现原理很简单,首先需要附加一张表记录每个用户的默认头像随机号,当然初始用户是没有该随机号的,用户通过该ID来寻找匹配的默认头像. 通过分析代码,所有的用户头像获取都是通过一个函数获取的 函数名为`public function getUserAvatar()`,那么我只需要在这个函数之前进行处理就可以实现了. 

那么如何处理呢? 现在想法比较简单,就是判断是否有该用户分配的 随机号,有就获取这个图片,没有就随机生成一个,然后存储,以后就继续用这个ID. 下面列出详细内容.

###3. 数据库增加

数据只需要三个字段,那就`自增的id`,`用户的id`,和`随机图片的随机号`. 数据表增加的SQL语句见下.

	--
	-- Table structure for table `ts_avatar_default`
	--

	CREATE TABLE IF NOT EXISTS `ts_avatar_default` (
	  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'avatar Id',
	  `uid` int(11) NOT NULL COMMENT 'avatar uid',
	  `avatarid` int(11) NOT NULL COMMENT 'avatar id',
	  PRIMARY KEY (`id`)
	) ENGINE=MyISAM  DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC AUTO_INCREMENT=12 ;

###4. 代码修改内容

代码修改主要集中于获取用户avatar的入口函数,应该是所有的用户头像的获取都是通过该函数完成(当然除了缓存外),所以只需要修改该函数即可. 一下是修改的代码,其中注释详细说明了代码的用途.

代码目录: /addons/model/AvatarModel.class.php

	//加入一个工厂类,用于模型化数据库
	//Xin Meng's fix
	/**
	 * factoryModel
	 * 工厂方法
	 * @param mixed $name
	 * @static
	 * @access private
	 * @return void
	 */
	 public static function factoryModel( $name ){
	 return D("Avatar".ucfirst( $name ), 'avatar');
	 		//ts_avatar_default
	 }
	//Xin Meng's fix end

	public function getUserAvatar() {
		$empty_url = THEME_URL.'/_static/image/noavatar';
		
		//xin meng 一下为修改内容
		// 首先判断 user 的默认图片id是否为0 如果为0 则初始随机生成1个数,
		// 否则直接 使用默认图片id
		// 获取数据库表 ts_avatar_default 查看是否有uid项目
		// 连接数据库
		$avatarDao = self::factoryModel( 'default' );
		// 查询映射
		$map['uid'] = $this->_uid;
		// 查询该_uid是否已经分配的初始头像
		$result  = $avatarDao->where($map)->find();
		// 释放查询映射
		unset($map);
		//如果查询结果为NULL,表示没有分配初始头像,那么开始分配(注,这里可以产看ThinkPHP的文档,寡欲GUID操作的返回值)
		if($result == NULL)
		{
			//制作随机数,这里范围根据你的随机图片数目而定,rand(1,100)代表有100张头像
			$rand_num = rand(1,100);
			//储存该随机值,通过数据操作,创建表项,
			$map['uid'] = $this->_uid;
			$map['avatarid'] = $rand_num;
			//插入操作
			$addId = $avatarDao->add($map);
		}
		else 
		{
			//如果查询结果不为NULL,说明数据库中存在该表项,那么将随机数值直接从数据库取出
			$rand_num = $result['avatarid'];		
		}
		//构造图片路径的字符串
		//参考 目录/addons/theme/stv1/_static/image/noavatar之下默认头像文件的命名方式
		//关于图片下一节详细介绍
		$str_big = "/big".$rand_num.".jpg";
		$str_middle = "/middle".$rand_num.".jpg";
		$str_small = "/small".$rand_num.".jpg";
		$str_tiny = "/tiny".$rand_num.".jpg";
		//构造初始头像
		$avatar_url = array(
			'avatar_original' 	=> $empty_url.$str_big,
			'avatar_big' 		=> $empty_url.$str_big,
			'avatar_middle' 	=> $empty_url.$str_middle,
			'avatar_small' 		=> $empty_url.$str_small,
			'avatar_tiny' 		=> $empty_url.$str_tiny
		);
		//原来默认头像分配初始化语句删除
		// 		$avatar_url = array(
		// 			'avatar_original' 	=> $empty_url.'/big.jpg',
		// 			'avatar_big' 		=> $empty_url.'/big.jpg',
		// 			'avatar_middle' 	=> $empty_url.'/middle.jpg',
		// 			'avatar_small' 		=> $empty_url.'/small.jpg',
		// 			'avatar_tiny' 		=> $empty_url.'/tiny.jpg'
		// 		); 
		
		//Xin Meng end 以上为修改内容
		$original_file_name = '/avatar'.$this->convertUidToPath($this->_uid).'/original.jpg';

		//头像云存储
		...

###5. 关于头像图片

默认的头像图片目录为`/addons/theme/stv1/_static/image/noavatar`,其头像图片种类有4种分别是

*. big		尺寸 200x200
*. midle	尺寸 100x100
*. small	尺寸 50x50
*. tiny		尺寸 30x30

我们的随机图片在此基础上加入随机数,譬如

*. big1.jpg
*. midle1.jpg
*. small1.jpg
*. tiny1.jpg

将你的随机头像图片按照上面的命名方法重命名后就 上传到对应的目录就可以使用了!

附注: 关于图片的处理,建议使用一些工具批量重命名,改尺寸和加水印. 如果你使用Mac的话,建议查看以下两篇文章.

*. [Automator工作示例-批量重命名](http://keben.diandian.com/post/2011-08-09/3698868)
*. [PhotoBulk要批量给图片加水印改尺寸](http://www.jbguide.me/2012/12/07/photobulk/)

###6. 实现效果

大家可以自行到[@导游|idaoyoo](http://www.idaoyoo.com)注册查看!

###7. 未来扩展内容

感觉当前图片比较杂乱,几百张图片随机挑选,总归还是有些瑕疵,后面可以考虑参考性别等,分类提供随机图片.尽量让用户不用上传头像,也可以得到一个比较满意的头像!

另外还可以考虑加入一个头像选择的功能,这个功能虽然比较普遍,但是加上去不容易...
