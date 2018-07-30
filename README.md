## jqxq
uci engine for chinese chess 翻看一下开发记录，竟然断断续续弄了这么长时间了，真是十年一梦啊。

![Alt text](电影与现实.jpg)

## 2017年7月13日

为了配合ncch，已有3处修改了：
1、makemove()里去掉了prefetch();否则出错，原因待查。
2、增加了sendmessage();发送自定义消息。
3、ucioption.cpp里增加了一个函数用于接受uci指令。
	
##2017年1月26日
	修改了ini类；修改了uci::init()函数，优先从配置文件中取Options。UCI：init()
	修改了endgame.h和endgame.cpp 不再需要dtm.cpp 文件了（主要是会导致编译后执行文件太大，会超过2M！）
	DTM分数直接在基类中返回。

##2017年1月20日
	1、开局库合并去重搞定（多出一条记录的问题搞定）；
	   支持jqxq.ini
	   配置文件（仅有少量参数用于测试，如控制各类开局库着法的查找，参数未整理完，
	   项目增加了3个文件ini.h ini.cpp及配置文件示例jqxq.ini）；
	   整理开局库代码，修订了查询策略：支持云库查询（支持代理服务器,项目增加了1个文件cloud.cpp）、
	   本地库及内置简易库，
	   所有查找到的着法均可组合起来然后随机输出一个，开局库功能够用了。
	2、2016年11月27日，完成残局评估相关工作，2017年1月18日，利用DTM残局库统计表数据自动生成dtm.cpp等代码，
	项目增加了1个文件dtm.cpp
	
	编译问题解决：文件太大了，修改：工程属性－配置－C++－命令行 面板中最下面增加 /bigobj 选项即可！！

	接下来:
	1、继续调试估值函数。
	2、调试多线程运行。

##2016年12月12日
	
   加上了版本信息，修正了特定残局的返回算法，增加了与位置相关的驱动走棋的评估信息?
裨蜃咦游薹较蛞迹?
	至此，才算是真正完成了 Ver 1.00 一二九运动纪念版 :)

2016年12月8日
	终于不用m_ChessManValue[2]及m_PositionValue[2]了，与之相关的代码也删除了！
	修正verify_material中错误！

2016年12月7日
	原evaluation->init()中把黑方位置分改为负数，现移动到position->init()中，黑方位置分变为负数，减少了do_move（）
	中更新st->psq时的判断颜色的两个if语句，提高了效率，构思还是挺妙的。
	把位置分合并计算并放入st结构中了，正确性调试完成，效率是否提高则未测试，do_move()中多余的与m_ChessManValue[2]
	及m_PositionValue[2]相关的代码未删除，把子力分合并计算并放入st结构中了，正确性待调试，多线程时assert
错！单线程能通过！但不用st->value就可以了，不影响程序正常运行。

	search中所有裁剪算法全部调试通过，可以打开，解出率非常高，但解出深度变深了，可能是参?
跫壬柚貌痪匪粒髡?
	开多线程可以完美运行，解出率也挺高，但效率有没有提高没有测试，有空慢慢测。
	测试用的典型局面增加到27个12月4日 search.cpp修改了一些参数，精简了perft相关的一些代码。

	1207 bench: release版 多线程效率确实高些
	1206 bench: release版 16194ms, 11440026 nodes ,706.436 kbps      bench 
多线程时assert错！单线程能通过！

2016年11月29日
	开局库相关：
	改了一处错误：position::init（）数组越界了 从0开始应该是<256
！一不小心就出错了。
	position::init（）中初始化随机数表时，增加了写进文件代码

2016年11月27日
	//增加知识型残局库的相关模块（material.cpp及endgame.cpp）
	//完成残局模块中的Key key(const string& code, Color c)函数。
	
	特定局面通过构造函数Endgames::Endgames()加进特定残局映射表，根据唯一的
material_key从特定残局映射表索引相应的评估函数，要增加一个特定局面的评估值，需
3个地方的修订：
	1、在enum EndgameType中增加对应的类型枚举值
	2、在构造函数Endgames::Endgames()中加进残局映射表
	3、增加评估函数体（用于该特定局面的模板评估函数）
	以上3部分内容都可以根据DTM残局统计表自动生成
	
	如果是要增加某一类局面，因其有多个material_key）,
按照特定残局映射显然不行了，必须按照以下几步来修订：
	1、在enum EndgameType
中增加对应的类型枚举值，此步与特定残局局面相同，且只能手动增加
	2
、在专用数组中增加映射关系，通过判断某一类残局局面确定该数组，然后再索引到相应?
钠拦篮瞬接胩囟ú芯志置娌煌一剐柙黾优卸虾庑┦橛牒?
material.cpp中定义
	3
、增加该类类型的残局局面的模板评估函数，此步与特定残局局面相同，且也只能手动增?
?
	
	//如果是要添加返回ScaleFactor
的特定局面或一类局面的评估，要求基本与返回评估值的步骤相同。

2016年11月23日
	查找为何update_stat()会有影响：
	该函数只是存储数据，应该是应用存储的数据时出了问题：在movepick.cpp
杀手启发着法返回时使用了先前存储的数据，先注释掉看看。
	注释掉以后，基本可以恢复update_stat()函数的正常使用，找ＰＶ时偶尔会出现assert
错。
	反复检查与countermoves及followupmoves
应用有关的逻辑都没发现问题。因此怀疑杀手启发返回的着法判断合法性时出了问题，
	修订了legal(Move m)
函数（增加走动的棋子不是当前走子方的判断）后问题消失，这也是一个潜伏较深的老问?
庖恢蔽捶⑾郑〈舜沃沼诟恕?
	
之前也发现了一个潜伏了许久的错误：调试过程中居然发现兵种编号顺序号中士象搞错了?
ㄇ狈诵砭玫拇砦螅。? 呵呵，bug无处不在啊
	解决上述潜在错误后，probcut也不会出错了，又多了个prune手段！但shallow prune 
有2个分数略有差别，其它都解对了！
	现在完全可以作为 cavalier Ver1.0 发布了　：）
	
	接下来
	1、位置分与子力分的设置问题（不改不影响准确性，但影响效果）
	2、move_loop里 Step 13. Pruning at shallow depth (exclude PV nodes)仍不准确？
	3、继续调试估值函数
	4、尝试多线程运行

	其它功能性工作
	2、引入开局库管理，内嵌简易开局库
	3、引入知识型残局库（完成一子库等简易残局局型）
	哇，还是有好多工作还做啊！

2016年11月20日
	１、屏蔽update_stat()
几乎解对所有测试局面，仅一些分数不准确及层数略多，原来是这个惹的祸！。
		extract_pv and insert_pv 中两个断言能通过了。马对单士解正确了！
		还有其它前面一些莫名其妙的问题都没了！

2016年11月16日
	查错（该改的都改完了，余下就是艰难的调试除错了！Oh My God!）
	return pos.side_to_move() == RED ? scores : -scores;   
//注释掉此句，竞然循环局解对了！但这不是原因
	注释掉 RootMoves[i].insert_pv_in_tt(pos); 也还是出现PV错误现象。
	extract_pv and insert_pv 中两个断言不能通过，原因待查。
	马对单士中出现长循环！

	修改：
	1、bool Position::pseudo_legal(Move m)const 中增加if(nSrc == nDst) return 
false;	//起止位置相同不合法，同时涵盖MOVE_NONE和MOVE_NULL
	2、is_draw()中增加了if (st->rule50 > 120)
	3、修改uci.cpp中的position函数以适应两个设置局面的命令
		“position fen 2bk2C2/4P4/2na5/3R5/p8/9/P8/2r2A3/3K2c2/5A3 b - - 0 8”			
//老版1.9兵河
		“fen 2bk2C2/4P4/2na5/3R5/p8/9/P8/2r2A3/3K2c2/5A3 b - - 0 8”					//3.6版兵河
	4、notation.cpp中score_to_uci（）新增为了适应兵河显示分数
	5、后来又发现修订了pseudo_legal
里的另一个错误（此是后话），走动的子不是当前该走一方的逻辑错误
	
	思考流程：
	go infinite -> go()->Threads.start_thinking(pos, limits, SetupStates)->main()
->notify_one();
	wakes up the thread when there is some work to do).
	接下来：sleepCondition.notify_one();cond_signal(c);SetEvent(x);   
	最后在主线程调用search::think();启动搜索

	注意：do_move()函数中，吃了子则st-psq应该加, sf50里是减，因为其预设值为负数。

2016年11月3日
	一、查bug，以下问题原因待查：
	1、马擒单士红先可解，黑先竟然解不出。
	2、一个停着杀局面解不出。是否PV检出代码有问题？
	3、深度大了，PV出现循环，有时出现错误（某方着法重复）与4中所述类似
	4、在do_move()中  st->psq -= WeiziValue[color][chesstype][dst]; //取消注释PV
不对但－9988分数对，其它杀局大部分解的也对，但以下"3a4r/4ar3/3k1P2b/9/9/9/9/9/
4AC3/4K4 w - - 0 1"  9991分数对，层数多了却又不对了，且PV
错误，这个可能是另外原因，待查。进一步检查，发现与循环测试中的允许重复次数的检?
獯氪胗泄兀渭鹯epititionDetected函数。
	
	二、其它功能性工作
	1、继续调试估值函数
	2、引入开局库管理，内嵌简易开局库
	3、引入知识型残局库（完成一子库等简易残局局型）
	4、尝试多线程
	哇，还差好多东西啊！
	
	三、已完成的一些辅助性工作：	
	1、清理了position.h:  比对了所有函数，所有数据私有了，增加了put_piece\
move_piece\remove_piece三个内联函数,删除了moves
数组，修改了循环检测相关代码并顺带解决了循环局面的返回值问题，删除了两个print(
)函数，重写了pretty()函数，可充分利用Log记录搜索日志。
	2、清理了square.h中多余的内联函数

2016年10月29日
	set_state()中st->checking = !Checked(sideToMove)? Checking(opposite_color(
side_to_move())) : 0;调试完成
	新增了fen函数并调试好，仅留下set,fen两个新增函数完全替代了原来的from_fen与
to_fen
	修正了set_state里的两个错误 st改为si 
	要去掉moves数组，只需新修订循环判断函数即可，这样两个大数组就完全可以不用了。
	
	去掉了chessmanvalue，position_value
	去掉了用于循环判断的变量m_MiniHash
	去掉了position.h中的m_NonEatSteps变量，全部用st->rule50替代.
	去掉了position.h中的m_hashKey64变量，全部用st->key替代.
	position.h中的m_Player改为sideToMove
	Key material_key() const  返回值已修改。

2016年10月18日
	为了代码简洁优美，新增写了一个函数： void Position::from_fen(const std::
string& fenStr )及配合函数void Position::set_state(StateInfo* si) const;
	为了强化残局的处理：position.h 中增加修订了以下成员变量与函数：
	1、uint8_t pieceCount[COLOR_NB][PIECE_TYPE_NB]; 
主要是有了兵种计数后，便于残局时处理不同的子力配置，增加知识型残局库。
	2、Key material_key() const 与新增的成员变量pieceCount有关。
	

2016年9月15日
	设计了一个估值函数测试盘面（含：当头炮，沉底炮及空心炮等棋型）
	//const char* StartFEN = "C3k1r2/3rn4/3N3P1/p3RR1PP/n3C1p2/6P1p/4cpp1P/9/9/
c1BAKA3 w - - 0 0";				//当头炮、沉底炮及空心炮等棋型
	//const char* StartFEN = "2bak1n1R/3ra4/3Nb2n1/p1p1CPp2/4c4/2P1CcB1R/P3P1P1N/
pp1KB3r/9/3A1A3 b - - 1 0";	//G4位保护判断
	//const char* StartFEN = "2bk1an1R/4a4/3nb2N1/p1p1CPp2/4c4/2P1CcB1R/P3P1P1N/
pp1KB3r/9/3A1A3 b - - 0 0";		//将帅牵制
	//const char* StartFEN = "C1bakarR1/3r5/3N3P1/p4R1PP/n3C1p2/6P1p/4cpp1P/4B4/
4N4/3AKA3 w - - 0 0";			//空心炮与窝心马
	//2bak1NRC/7c1/7rb/p7n/4C1Rnr/9/9/3Nc4/4A4/3K1AB2 b - - 0 5													
//各种牵制目标车
	开始调试eval命令：
		双方预评估完毕后才能比较某一区域的双方力量对比，因此相关代码要移出、移到
evaluata_init之后。
	调试过程中居然发现兵种编号顺序号中士象搞错了（潜伏了许久的错误！）

2016年8月30日
	0、在position.cpp里增加了保护判断函数::protected();
用于判断特殊牵制棋型时判断牵制目标子是否存在保护，已完成未调试。
	
	1、evaluate_init
（）为局面预评估函数。约定盘面红下黑上！先断言不会被将军（需做到进入前总是解将?
卸掀寰纸锥危紊枋占骼嗥遄酉嗷デＶ菩畔ⅲ偕韪髌遄幽艽锏降奈恢茫纱?
到格子、捉住或威胁某个棋子即威胁与保护属性），顺带完成灵活性评估与九宫区域控制?
拦馈?
	//1、判断局势处于何种阶段，开中局还是残局阶段（计算各种棋子数量，按照车=6
、马炮=3、其它=1相加），根据不同阶段做些必要的参数或分数值调整。
	//2、判断各方进攻防御状态，分红左红右及黑左黑右4
个区域统计，判断是否存在多子归边攻势（计算过河子数量，按车马2炮兵1
相加），根据进攻还是防守状态做些必要的参数或分数值调整。
	//3
、扫描收集红黑双方对将帅的牵制信息，填充被牵制子信息结构，用于判断是否可移动及?
云渌遄拥谋；せ蛘咄彩欠裾媸涤行Вㄈ绻τ诒磺Ｖ谱刺贫鞘芟薜模叶云渌?
棋子的保护与威胁是虚假无效的）。
	//4、扫描所有棋子可移动情况，填充ei
中有关的威胁与保护（与棋子相关）、灵活性及控制棋盘区域（与棋格或棋盘区域相关）?
挠泄匦畔ⅲ院罂捎糜诟髦制拦馈＃ū静街柚惺欠褚萸Ｖ菩畔⒐说舨荒芤贫钠宀?
，过滤掉因被牵制可保护或威胁实际上却起不到保护与威胁作用的无效棋步？）。
	//5、检查预评价是否对称（实际每一个评价要素都应该有对称是否的检查）
	
	2、王安全性评估为核心评估函数。主要包括以下四大控制和进攻势力：
	2.1、评估王与其它棋子位置关系在攻势上的体现
	//2.1.1、炮的攻势评估	//1、当头与沉底空心炮	//2、窝心马	//3、沉底炮	//4
、当头炮 
	//2.1.2、王被牵制评估	//1、车炮对王的牵制	//2、马对王的牵制	//3
、将帅互相牵制情况
	2.2、评估棋盘关键区域子力平衡与控制及多子联合攻势
	//2.2.1
、九宫控制力（此部分可提前至预评价模块中棋子扫描时棋格或棋盘相关的评价部分）
	//2.2.2
、区域子力平衡，识别多子归边攻势（此部分可提前至预评价模块中判断进攻防御状态时?
恚?
	
	3
、特殊棋型评估函数，评估双方主力棋子相互牵制情况，以对方车为牵制目标，主要包括?
韵录钢值湫偷那Ｖ评嘈停渌Ｖ评嘈驮莶豢悸牵?
	//1、车对车炮	//2、车对车马	//3、炮对车马	//4、马象弱牵制	//5
、车对炮兵、马兵的弱牵制
	
	4、马车灵活性评估，已完成未调试（此部分提前至预评价模块中）。
	
	5、单兵种评估共6个，已完成未调试。
	
2016年8月10日
	
一不小心居然输给了它！呵呵，功夫不负有心人，终于出成品了。以后就只要考虑如何提?
咂辶α耍ㄖ饕楦慕缶郑Ｔ俅嬉环荩?

2016年8月9日
	fen rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR w - - 0 1 
moves h2e2 c6c5 h0g2  兵河发出的局面信息，与UCI标准协议不一致？没办法，改吧

	经测试，如果注释掉第9和13步，则可以解那个循环局，分数对。只加上第9步probcut
则解不出来，只加上第13
步解不完全准确，都加上则解不对那个循环局，但可以解那个江湖局，不过要27
层！。进一步测试，如果注释掉第9和13步，基本OK了。
	存一份先！！
	下一步：
	0
、整理一下搜索函数的编写经验。整理一下还需要那些控制变量（控制搜索等程序行为用?
嘈磇ni文件作准备）
	1、查找兵河加载后不能正确运行的原因（810已修正，可以加载了）。
	2、尝试添加审局！
	3、着法评分部分有待完善，see函数还遥遥无期！
	4、编写简单的加载测试UI。
	5、下一步为并行解题作准备：
		*试着去掉position结构里面的2个大型数组及冗余的变量，。
		*改编为多核心版本。
	6、将军步生成函数未测试（有重复着法？），其中一个解将函数不完全正确，
	仍然还有好大的工作量啊！
	
2016年8月8日
	search模板重新对照sf50来做了一遍，要修改的地方都加了注释  "modify" 
	修改了MOVE_NULL定义，原来是0，现改为0xffff.
	去掉了ProbCut（主要是还未搞懂原理）
	去掉了不必要的legal(move).
	bench
都没问题，但解杀局不全准确，有的分数不对，有的分数对了层数又不对，待进一步检查?
魇浴?
	用searchold则注释掉extract_pv()里面的条件legal(move)
可以解那个循环测试局，但分数仍然不准。
	可以 go infinite.
	下一步试着去掉position结构里面的2个大型数组：
	
2016年8月3日
	今日版本可以存一下档了（基本框架已完成了）	
	
使用模板技术重写了走法产生器（吃子、非吃子及全部。解将及生成将军步函数未修改）?
氪蟠蠹蚧唷Ｐ掳娉宰印⒎浅宰蛹八凶叻ú魍ü馐裕硗庥衷黾恿艘桓鱿?
应的新版但未调试好）。穷举法解将通过perft
测试，另一个解将法则未通过（与穷举法解将数目不一致，待调试），将军步生成器未测?
酝辍?
	
现程序结构，走子之前是否将对方比较麻烦，需另外编码，如果提前走子测试又会与后面?
淖咦勇呒馗矗跋焖俣龋魏危?
	似乎静态搜索函数里面无需循环检测：理由如下：…………
	重新调整分数，保证任何时候搜索返回值都会小于VALUE_INFINITE
	修改了position::pesudo_legal(Move m)和legal(Move m)函数。
	去掉了ChessID类型的应用（此类型仅作解释问题用）。

	修改许多其它代码来适应SF5
。全部头文件中除与实际逻辑紧密相关的几个外，以下均可沿用sf50
的源文件，完全不用修改：
	material.h	完全不用修改（暂时无相关的.cpp文件，用于处理子力等，在thread.h
中，可注释掉）
	pawns.h		完全不用修改（暂时无相关的.cpp文件，用于判断兵形结构，在thread.h
中，可注释掉）
	bitcount.h	完全不用修改（暂时也不需要)
	platform.h	完全不用修改
	endgame.h	完全不用修改
	misc.h		完全不用修改
	movegen.h	完全不用修改
	book.h		完全不用修改
	ucioption.h	完全不用修改
	rkiss.h		完全不用修改
	tt.h		完全不用修改
	timeman.h	完全不用修改
	evaluate.h	完全不用修改
	search.h	完全不用修改
	movepick.h	几乎不用修改 （加个T修饰下table返回值就可以了）
	notation.h	几乎不用修改 （删除第2个形参：bool Chess960。如果增加const修饰
Position& pos更佳（对应cpp里也要改） ）
	thread.h	完全不用修改 （目前也可注释掉以下几行：pawns.h、material.h、Material
::Table、materialTable、Pawns::Table pawnsTable及Endgames endgames）
	以下几个要根据实际情况来编写（position.h还有进一步修订趋同的余地）：
	types.h piece.h square.h position.h及bitboard.h

	模块文件中，以下文件完全不用修改或几乎不用修改即可用：
	timeman.cpp     完全不用修改 
	ucioption.cpp	完全不用修改
	misc.cpp		完全不用修改	
	tt.cpp			完全不用修改
	thread.cpp		完全不用修改，另外一个与多线程有关的函数在search.cpp中，void 
Thread::idle_loop();
完全不用修改，是否意味着是多线程相关的代码都不用修改？（未仔细确认调试）
	benchmark.cpp   几乎不用修改（3处小改：1、char* Defaults[] 当然要改 2、fens.
assign(Defaults, Defaults + 16);第二个参数要与defualts
中的条目匹配数量，否则出错 3、删除Chess960相关）
	uci.cpp			几乎不用修改（1、const char* StartFEN 2、loop()里调用position()
函数条件要改position->fen. 3、position()小修改。4、perft命令中：depth = (is 
>> depth)? depth : 0; 5、可在loop
（）中初始化初始局面后加入一段调试用的显示代码。
	movepick.cpp	几乎不用修改（仅修改评分函数。需增加一个比较函数(731
后改得用不着了)。
	endgame.cpp		只保留了一些模板，可通过编译，其它残局内容待增加
	main.cpp		修改很少，要注意各模块初始化工作的安排。
	position.h		因为具体逻辑不同，无法改成一样，但search.h
头文件已修改得完全一致，没有任何区别。

	search.cpp、position.cpp文件为核心模块：
	search.cpp	中一些用于搜索的数据表要根据实际修改。search函数中要判断do_move
的成败！要增加循环判断！
	search.cpp	中以下几个函数完全不用修改或几乎不用修改： 
	void Thread::idle_loop();	完全不用修改		
	void check_time();			完全不用修改		
	void Search::init();		完全不用修改		//只是数据可能不准确或适应性差
	void Search::think();		完全不用修改		
//如需输出格式改为中文，而可在最后略有修改，仅用于调试版本方便阅读。
	//以上是接口函数，以下是内部函数
	void update_stats();		几乎不用修改		//仅一处修改：-bonus 要强制转换为Value
类型。
	void id_loop(Position& pos) 几乎不用修改		
		//1、新增一行(ss-1)->ply = pos.game_ply() - 2;	不改则分数少1分。
		//2、两处std::max,min
函数在枚举类型参数前加个“+”就可以了（类型自动转换了？）。
	匿名空间中声明变量及函数原型部分：
		1、修改了两个边界函数razor_margin和futility_margin；
			inline Value razor_margin(Depth d) { return Value(VALUE_ROOK + VALUE_PAWN 
* d); } //d=3、2、1		//修改
			inline Value futility_margin(Depth d) {   return Value(VALUE_CANNON * d);  
}						//修改
		2、为了增强perft函数而增加的数据类型说明及函数声明
		//以下为了增强perft函数而增加的数据类型说明及函数声明
		struct CPerftResult
		{
			size_t totals;
			size_t checkers;
			size_t captures;
			CPerftResult():totals(0),checkers(0),captures(0){;}
			//CPerftResult& operator=(const CPerftResult& obj)
			//{
			//	if (this != &obj)
			//	{
			//		totals= obj.totals;
			//		checkers= obj.checkers;
			//		captures= obj.captures;
			//	}
			//	return *this;
			//}
			CPerftResult& operator=(const int& obj)
			{
				totals= obj;
				checkers= obj;
				captures= obj;
				return *this;
			}
			CPerftResult operator+(const CPerftResult& obj)
			{
				CPerftResult temp;
				temp.totals = totals+ obj.totals;
				temp.checkers = checkers+  obj.checkers;
				temp.captures = captures+  obj.captures;
				return temp;
			}
			void operator++() 
			{ 
				totals++;
				//checkers++;
				//captures++;
			}

			void operator+=(const CPerftResult& obj)
			{
				totals += obj.totals;
				checkers += obj.checkers;
				captures += obj.captures;
			}
		};

		inline std::ostream& operator<<(std::ostream&os,const CPerftResult& obj ){
			os<<" totals: "<<obj.totals<<" checkers: "<<obj.checkers<<" captures: "<<
obj.captures<<std::endl;
			return os;
		};

		template <typename T> static  T perft(Position& pos, Depth depth);
		CPerftResult perftPlus(Position& pos, Depth depth) ;

2016年6月29日
	整理代码，使头文件尽量保持一致（现仅types.h  position.h  bitboard.h
三个头文件相差较大，其余完全不用修改或几乎不用修改）
	删除了“history.h”与“history.cpp”两个文件
	
删除了走法生成器里与分数有关的代码，仅生成伪合法走法，不再生成分数部分（走法生?
珊笤趐ickmove.cpp里统一评分）。
	如此走法生成代码应该还可以简化。
	可以顺利完成bench任务。下一步对照检查position.h文件，重新检视search.cpp
	更新了函数：void update_stats(const Position& pos, Stack* ss, Move move, 
Depth depth, Move* quiets, int quietsCnt)

2016年6月28日
	不开多线程，可以解杀局了！先存一份。
	可以go infinite了。但结果不准确，待细调！
	perft测试准确！
	
注：单机版中的着法生成器有不够完善的地方，如果要用，应该移植多机版本的过去。后?
研拚╯elfkilled一个变量未初始化错误）
		仍有checkingGen函数中存在变量未初始化问题待查，但这个函数不是必需的。

2016年4月28日
	2016年重新用VC2010编译时，出现以下问题： 
	vc 2010 学习版要注册：用以下注册码解决了问题 6VPJ7-H3CXH-HBTPT-X4T74-3YVY7
	连接时出现：“LNK1123: 转换到 COFF 期间失败: 
文件无效或损坏”错误信息，从这里“http://blog.chinaunix.net/uid-20385936-id-
3506149.html”找到解决方案：
	连接器LNK是通过调用cvtres.exe完成文件向coff
格式的转换的，所以出现这种错误的原因就是cvtres.exe出现了问题。 
	在电脑里面搜索一下cvtres.exe，发现存在多个文件，使用最新的cvtres.exe
替换老的文件即可，替换之前记得备份一下，如果不对，可以替换回来。 
	例如：我的电脑里面安装了vs2010,
最近更新了系统，打了一些补丁，结果就出现这种错误了。在电脑里面搜索发现 
	C:\Program Files\Microsoft Visual Studio 10.0\VC\bin 
	C:\Windows\winsxs\x86_netfx-cvtres_for_vc_and_vb_b03f5f7f11d50a3a_6.1.7600.
16385_none_ba476986f05abc65 
	C:\Windows\Microsoft.NET\Framework\v4.0.30319 
	这三个路径里面都有cvtres.exe
文件，于是我尝试使用第二个路径里面的文件替换第一个路径的文件，问题解决。 
	参考资料如下： 
	http://stackoverflow.com/questions/10888391/link-fatal-error-lnk1123-failure-
during-conversion-to-coff-file-invalid-or-c/14144713#14144713
	LINK : warning LNK4075: 忽略”/EDITANDCONTINUE”(由于”/INCREMENTAL:NO”规范)
这个问题是因为在vc6中，工程使用的增量编译。
	解决办法：属性，链接器，常规，启动增量链接　选择　是(INCREMENTAL)或者 
选择项目 属性->配置属性->c/c++ 修改调试信息格式为 程序数据库（/zi） 


/*两年以前*********************************************************************
****
20140509
	完成了MoveList类的接口代码；
	计划movepick类利旧，待调试
	修改了，position.do_move
函数，待调试（写一个比较走子前后的比较函数用于调试比较方便）

20140427
	尽可能靠近stockfish_DD。
	修改了走法产生器接口代码及棋盘代码。可以执行了。但崩溃 :）

20130108
	电脑升级了，终于能用VC2010之类的大块头了。带来的好处也很明显：
	1、不再像VC2005环境一样要另外安装SDK。仅需要安装VC2010学习版就可以了
	2、在VC2010学习版中利用现有C与H文件，通过建立空的windows控制台项目就可以了。
	3、编译速度大大加快，yeah！
	4、去掉了一些不必要的文件。
	5、工作太累，时间精力实在有限，多线程程序没有实质性进度。以后抽空改进。
	附：VC2010学习版注册码：6VPJ7-H3CXH-HBTPT-X4T74-3YVY7

20100905
	多线程的程序也终于可以解残局了。
20100818
	放弃单线程的开发，参考sf作多线程思考
	修正了将军步产生器里的一些错误，过滤掉了重复着法


*以下为单线程的开发过程，暂不更新了********************************************
****
2010年4月25日：
	对照stockfish1.7修改了些注释，增加了些搜索方面的内容。
	VC2005中不支持strcasestr函数，另外增加了一个StrStrI_ljh替代功能，没有测试。
开发历史: 
2010年4月3日：
	5a3/4k4/4bPn2/9/3rPnb2/4C4/9/3R5/4pp3/3K1p3 w - - 0 1解不出来
2010年3月28日：
	修正了时间错误，propertimer 与 limittimer单位为毫秒，总用时数，为绝对数
	maxtimer 与 mintimer为相对时间，即为某一时间点，为相对数
	找到内部深度迭代时-2出错的错误原因，移动了延伸代码至其后。
	完成了ini文件读写类
	完成了“象棋路边摊”的测试代码。
	10层40秒，11层46秒，12层67秒
2010年3月19日：
	
穷举法解将函数生成的着法做自杀式检查，进步过滤了非法着法，移将解将则未作将军检?
椤?
	4P1PP1/3k4C/2rc5/5N3/2b3b2/9/9/9/9/1rnK1CRc1 w - - 0 1 有错误，待查
	修正了一些错误
	着法中文表示中前兵平三没前字    //已更正错误，原来是IsSameSide()有误
	新的穷举法解将函数时出问题：2b1k4/5c3/9/9/9/9/3n2P2/9/9/5K2R w - - 0 1 
错误已除，解炮马将时代码出错
	在PVSearch里作迭代加深时，依然出现内在读写错误!  取消或者 depth-3 
则无。本错误自然消失，原因不明
	把返回值 由 ply+1-winscore 改为 ply-winscore
2010年3月15日：
	
穷举法解将函数生成的着法做自杀式检查，进步过滤了非法着法，移将解将则未作将军检?
椤?
	修正了一些错误
	着法中文表示中前兵平三没前字
	新的穷举法解将函数时出问题：2b1k4/5c3/9/9/9/9/3n2P2/9/9/5K2R w - - 0 1
2010年3月9日：
	增加了穷举法解将函数，需要作将军检查，未调试。
	临时把解将函数用生成所有着法（吃子+非吃子方式）的方式，搜索速度提高很多。
	10层7、15秒，11层46、64秒，12层87、505秒。到14层883秒。
	证明原来的解将函数效率很低，需改进。
2010年1月3日：
	QS 哈希表分离开。
2010年1月1日：
	修正了一些错误。
	10层9.9秒，11层55秒，12层85秒。
	在PVSearch里作迭代加深时，依然出现内在读写错误!  取消或者 depth-3 
则无。原因难找！ 但速度加快，13层230秒

	QS里部分着法无需作将军检查，比如：hashmove，解将着法，可节省一些时间。

2009年12月31日：
	双层哈希表已完成，开局11层24秒,12层339.64，13层433.28。兵河1分钟13
层相比，还有差距！
	
	发现HashMove有非法现象！这个比较麻烦。不好查。
	KillerMove经过合法性检测后仍然出现非法现象，需调试检查。
	以上两种情况，2010.1.1.找到原因，原来是写入哈希时，棋步写错了。现在hashmove
不用再检查合法性了.  assert(hashmove valid!);
	在PVSearch里作迭代加深时，出现Stack overflow!  
取消则无。可能递归太深或搜索函数里局部数据太多太大。
2009年10月14日：
	经初步测试：
		QS里不用Hash也不慢
		QS里不解将，不生成将军步
		1分钟可达到11层了 ：） 50.672秒
		PV路线有中断和不对的现象，有些局面解不出来！
2009年10月13日：
	PVS及ZWS增加历史裁剪，历史裁剪不完善。
	开局10层10.688秒，11层70.469秒 
	未作全面调试
	本次完成后，搜索引擎基本可用。存档！！
2009年10月11日：
	尝试加入无用裁剪和Delta裁剪
	修改了NULLMOVE的条件，但PV路径出现丢失现象
	下一步准备优化历史裁剪
2009年10月4日：
	国庆期间进行了重大改进：
	1、程序结构进行了较大的调整：PVSearch 与 ZWSearch 分开；CBoard继承了CHistory
；增加了CMoveSort类，各搜索代码更简洁
	2、发现了FromFen函数里的一个小问题，tempfen数组开小了，对于比较长的fen
串会出问题
	3、修正了QSort函数分数排序而着法未跟随的问题
	 、修正了杀手合法性判断里的一个错误
	3、下一步工作：
	3.1历史表红黑分开，搜索完成后保存，着法生成时获取(已完成)
	3.2简化杀手着法的应用，杀手着法固定为两个，不再支持多个杀手，这样代码更简洁，
2个杀手应该足够了。(complete!)

	3.3以下两种情况出现吃将帅的问题：找到QS里探索Hash出现非法Move
的问题；将军步生成器兵被对方炮牵制后移动有问题
	3.4优化历史剪裁及增加无用剪裁
	3.6闲时测试将军生成器及将军步生成器
	3.7搜索引擎稳定后增加估值部分
	3.5双哈希及QS里哈希单列
	置换表的覆盖策略通常有两种：深度优先和始终覆盖。
	参考中国象棋程序“纵马奔流”的做法，采用双层结构的置换表。 
　　双层置换表的第一层使用了深度优先策略，第二层使用了始终覆盖策略。
	记录置换表时，如果第一层没能覆盖(因为深度太浅)，那么覆盖第二层；
	读取置换表时，如果第一层没有命中，那么读取第二层。
	当第一层置换表被覆盖时(由于深度足够)，原来被覆盖的表项就保存到第二层。
	
换句话说，无论深度如何，写入置换表的深度总会和第一层比较，深的留在第一层，浅的?
环湃氲诙恪?
	50秒左右终于可以到9层了，但与兵河1分钟13层相比，差距还是太大！！
2009年9月20日：
	终于找到了几个重大错误:
1、	IID用错了
2、	杀手着法参数错了
3、	IID应在NULLMOVE后
2009年9月19日：
	取消了原来的归并排序法,快速排序方法调试成功.
2009年9月12日：
	历史启发类成员增加了快速排序方法（move ordering）
2009年9月5日：
	对程序结构进行了一些调整。

2009年7月19日：
	08年完成的版本，1分钟可到8层，基本能用。但速度与高水平的UCCI引擎相比差距较大，
	经多次修改，进步不大，且在目前的大的框架下已没有多大的进步空间。
	因此，本次重新开工，决定作较大的修改。
	1、修改HASH表为双层HASH表，并在QS里也增加HASH表的应用。
	   hash表项变更并增加项目：尽量存储较多的信息，同一节点的alpha值、beta
值同时存储（如有），也保存将军对方的信息。
	   最大限度地避免重复计算。
	2
、着法生成方面的修改，修订同时生成吃子与非吃子着法产生器，搜索时吃子与非吃子不?
俜挚峭鄙桑蔚谒阉鳌?
	3、启发信息修改：主搜索例程中，被将时只历史启发，不被将军时，MVV/LVA
＋历史启发。
	4、QS增强：
	   QS里被将时历史启发；不被将军时，空着启发＋MVV/LVA
，将军对方步限制深度不作启发并放在最后搜索。
	   QS
搜索直到没被对方将军并没有吃子着法为止，此时局面相对宁静，估值相对要容易而准确?
?
	5
、历史分数的修订：不好的着法（被将死的着法或非法着法）减少得分，这个对性能的影?
烊绾涡枰馐?
	6、考虑到将来硬件的发展（CPU
速度及存储空间都将加快加大），搜索尽量避免降低准确度方面的算法，
	   尽量以空间换时间（如以上双层及QS里HASH的应用就体现这一原则）。
	7、ROOT的特殊处理
	8、增加初步估值（除子力位置与价值分外）如
	9、棋盘结构里取消攻击子数目变量，取而代之以棋子位变量，方便估值的应用。(完成)
	10、SearchFromBook移至CSearch类里比较合理(9月2日完成)
2008年12月15日开始测试：
	修正了一些错误，包括生成马将军步里一个assert错误。
	QS里增加了将军步及解将步。但限制了深度。
	基本成形。但仍有个别局面解不出来。如：2b1k4/5c3/9/9/9/9/3h2P2/9/8R/5K3 r - - 
0 1  车兵对炮马象（较难的）
2008年12月10日开始测试：
	第一节点返回率不高，可能原因，节点排序不好，估值函数不准确
	时间控制不准确,每步？秒
	因为估值函数不准确，因此不识空头炮，马的灵活性未体现。王安全认识不足。
	上层较慢，开局一般8层。9层很吃力。

	以下bug待查：
	3k1ab2/R2n2N2/3ab4/p3p3p/2p6/9/8P/2c1BK3/3rA1p2/3A5 w - - 0 1 moves e1d2
	黑方走棋！搜索深度：7
	开始搜索......
	info depth 1 score   910
	pv info depth 1 score  9999 pv
	info depth 1 score  9999 pv
	info message 在规定的深度内，遇到杀棋！停止思考！
	nobestmove
2008年12月02日更新：
	初始化，搜索函数里的局部变量score,及修改了存放hash探测结果的变量，不用score用
hashvalue.如此返回值不再异常
	修改了NULLMOVE裁剪的窗口，原来为-beta,-alpha,改 为-beta,-beta+1
的这、零窗口，搜索应该加快一些。
	修正了上次判断alwayscheck里alphabeta搜索语句。
2008年11月25日更新：
	修改了CBoard类里 FromFen函数里的一个错误，FromFen里应用到了checking
函数，但之前CPreMove未初始化，因此不对。
	故把CBoard
类的构造函数里的代码全部移至搜索类里。着法产生器作为搜索类的一个类成员对象，先?
跏蓟恕Ｈ缓笾葱兴阉骼嗟墓乖旌牍什辉俪龃怼?
	
但这样从逻辑上看总觉得不舒服。因此类层次设计不合理。设计逻辑虽不好看，但代码总?
悴换岢龃砹恕?
2008年11月23日更新：
	增加了解連殺局選項.
	//以下判断是否应该连将对方.用于解连杀局
	//1、判断起始局面是否被催杀
	//2
、如果起始局面被对方催杀，所有不将军对方的着法都被杀，则必须连将对方才有希望
	//3
、如此判断是否是连杀局时，对某些非连杀局有影响，即将被将死时，最后一步棋有问题?
Ｈ?bk2C2/4P4/2na5/3R5/p8/9/P8/2r2A3/3K2c2/5A3 b - - 0 82
	修改了历史记录类,改为一维数组,并作了溢出处理
	QS里增加了hash应用.但没有单独的hash表.冲突率较高.
	QS里增加了杀棋步,有利于解连杀局,但速度变慢.
	QS里搜索到的杀棋如果返回杀棋分数则有以下影响:返回分数不准确,PV
路线后面的断了.解杀局则快了.
	重新整理了QS代码.
2008年11月16日更新：
	修正了仕不吃子的错误（产生器里B_BISHOP改成B_BISHOP1,产生器里R_BISHOP改成
R_BISHOP1）
2008年11月16日更新：
	在根节点里恢复历史裁剪
	在宁静搜索里恢复保存搜索结果
	经实验，杀手剪枝率比hashmove大，搜索顺序作了调整：killermove 优先搜索。
	对历史裁剪和NULLMOVE的条件作了些修改
	检测重复的杀手速度太慢，须改进
	被将军解将后，搜索时过滤掉hashmove and killermove.
2008年11月3日更新：
	在FromFen函数里增加了循环检测、检测长将及设置我方禁手。
	中断返回更准确
2008年11月2日更新：
	修正了循环检测里检测长将的一个错误。
	去掉CChessMove的构造，析构函数，速度快了不少，开局7层47秒。8
层耗时太多，无实战意义
	QS里也增加了MVV－LVA启发（没有保护判断）
	支持UCCI协议的代码作了修改
	此版本基本成型。可以战胜一般业余选手了！举杯庆贺！存档！20081103
2008年11月1日更新：
	修改了LookUpHash函数里一个非常弱智的错误（原来将军局面取值时居然修改了表中的
hashvalue）
2008年10月30日更新：
	静态搜索吃子着法时，只搜索吃车马炮的着法，否则节点太多
	开局终于可达8层了，用时4.5秒。裁剪率如下：杀手86.9% HASH 72.6% NULLMOVE 45.6%
	可见杀手着法裁剪率比HASH着法还高些。
	如果要到9层，还需努力。主要问题是
	1、空着裁剪节点太少。
	2、HASH表冲突厉害。
	3、各种裁剪算法的条件还有待完善
2008年10月23日更新：
	增加了另一个随机数产生函数
	在QSearch
里增加了生成将军步函数，如此解连杀局有利，但静态节点数目大增，可以考虑限制层数?
种平诘闩蛘汀?
2008年10月22日更新：
	修改了随机函数产生器，用RC4密码流产生随机数
	评估函数增加先手分
2008年10月21日更新：
	FromFen函数开始，增加了m_TotalSteps的初始化0.
	移植了eleeye最新的ucci协议实现代码（主要是base.h  base2.h parse.h pipe.cpp及
ucci.cpp等五个函数）
	仅去掉了parse.h里 的 #ifdef _MSC_VER 伪指令。
	修改了主程序文件以便匹配最新的 ucci 协议。
	对着法产生器的运算次序加了（），少了些警告信息

	经测试，
	1、第一次运算没问题，第二次就简单地返回9999
	2、有时一个深度耗时比预计的要多。
	估计问题如下：1、中断返回有问题 2、可能进入长时间的循环，循环检测有问题
2008年10月18日更新：
	又懒了一年，08年国庆后，得闲改正了一些错误
2007年11月8日更新：
	基本调试完成解将及生成将军步函数
2007年11月7日更新：
	更正了不少错误
2007年11月4日更新：
	1、部分完成生成将军步着法产生器
	2、下午开始调试
2007年11月3日更新：
	1、增加生成将军步着法产生器（没有经过严格测试）
	2、下午更新
2007年11月2日更新：
	1、增加解将函数
2007年10月28日更新：
	1、更改了数据结构，着法格式及棋子编码更加简洁高效
	2、采用位行、位列技术分步生成着法，吃子与非吃子分开搜索
	3、更改了历史启发算法
	4、着法顺序调整算法更新
	5、将军检测顺序调整
	6、优化了部分代码，速度更快
2007年以前（略）

验证招法生成器正确性，perft结果：合理着法的个数。(
多核版的生成器用原始盘面测试通过！perft 1-5数目都对！就是速度有点慢)
-----------------------------------------------------------------------------
----------------------
rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR w
     depth     nodes    checks            captures
         1        44         0                   2
         2      1920         6                  72
         3     79666       384                3159
         4   3290240     19380              115365
         5 133312995    953251             4917734
perft(4)  nodes: 3290240		time 29363		nps 112053（ljh）  
perft(6)  nodes: 5392831844		time 135643		nps 39757538           
没想到这个增长这么快。多一层差不多要多50倍的时间。
7层估计今天出不来，照这增长速度，10层不要几十年？
-----------------------------------------------------------------------------
---------------------
r1ba1a3/4kn3/2n1b4/pNp1p1p1p/4c4/6P2/P1P2R2P/1CcC5/9/2BAKAB2 w
     depth     nodes    checks            captures
         1        38         1                   1
         2      1128        12                  10
         3     43929      1190                2105
         4   1339047     21299               31409
         5  53112976   1496697             3262495
perft(5) nodes:53112976 checks:1496697 cap:3262495	time:460995ms nps:115213（
ljh,用穷举法解将及生成所有函数）
-----------------------------------------------------------------------------
----------------------
r2akab1r/3n5/4b3n/p1p1pRp1p/9/2P3P2/P3P3c/N2C3C1/4A4/1RBAK1B2 w
     depth     nodes    checks            captures
         1        58         1                   4
         2      1651        54                  70
         3     90744      1849                6642
         4   2605437     70934              128926
         5 140822416   3166538            10858766
其它网友的数据：
如果风云剑的数据是正确的..3个盘面各跑5层需要 74秒的话...
我不敢po文了..出乎意料..
三个局面各跑完5层..合计花的时间是... 5.6 秒
三个盘面各跑完6层，合计花的时间是 3分20秒24
没想到我X年前发明的的演算法竟然真的是个怪物..

对SF中Hash实现算法的一些疑问？
So my question is: how should I access the transposition table in such a way 
so that multiple threads can be doing searches, 
or is this even possible? I see that Stockfish's TranspositionTable class 
makes no mention of thread safety. 
Additionally inside Stockfish's search method, calls to TT.store aren't 
wrapped in a lock. 
So I'm a bit confused as to how Stockfish ensures integrity of the 
transposition table.
http://www.talkchess.com/forum/viewtopic.php?start=0&t=41917 

Basically there exist 2 'solutions' out there avoiding locks. 
Firstly (like done in Stockfish), validate the information presented by the 
transposition table, 
not to crash on entry corruption. 
Secondly (eg. like done in Crafty) you can xor the data of the entry back 
into the key 
in order to invalidate the key of any corrupted entry (see http://
chessprogramming.wikispaces.com/Shared+Hash+Table#Lockless).
http://www.talkchess.com/forum/viewtopic.php?topic_view=threads&p=563615&t=
51755

Also Stockfish does not use the "lockless hashing" technique found in Crafty, 
involving an XOR of the hash code with the value, 
nor does it use an actual lock. So the assumption also seems to be made that 
updates to the hash table will be atomic. ?

Counter Moves History
A combination of the History Heuristic in conjunction with the Countermove 
Heuristic, proposed by Bill Henry in March 2015 [6] , as already used by á
lvaro Begué in his Checkers program 20 years before [7] , was implemented by 
Stockfish contributor Stefan Geschwenter, further tuned and improved by the 
Stockfish community, and released in Stockfish 7 in January 2016, dubbed 
Counter Moves History and mentioned to gain some Elo points [8] . Stockfish's 
History and Countermove arrays are piece type and to-square based, not 
butterfly based. Each entry indexed by a previous move, is a complete history 
table with counters indexed by the move refuting that previous move.

A Refutation Move in the context of search algorithms like Alpha-Beta or 
Principal Variation Search is a move which fails high at Cut-nodes. It is not 
necessarily the best move, but good enough to refute opponents previous move. 
Since it is the best move found so far, in the context of transposition table
, it is often flippant mentioned as best move as well.
