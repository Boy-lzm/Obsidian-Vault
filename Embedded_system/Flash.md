**基本特点**
1. 只支持erase和program，program只能从1到0，erase是从0到1
2. program之前必须要先erase，不然会导致ecc错误，读出和写的不一样
3. PFlash 读取缓存保存的是上一次读取的数据，在擦或写之后这个buffer必须要被清掉，可以通过PFLASH_FCNFG[DATA_BUFF_CLR_EN]来使能自动清除。当读取数据后，再执行故障注入，必须要先读一下其他位置，再读故障的位置才会生效。
4. IFR：独立的flash内存用来保存参数

**术语**
1. Flash phrase：16 bytes的flash内存，16字节对齐，0x00000010,0x00000020...., 是flash中一次操作中最小的可编辑内存


**AB bank**
	用来切换flash0和flash1的地址映射关系
1. 当flash0或flash1为激活bank时，该flash bank的地址将会被映射到0x00000000-0x000FFFFF,该bank的IFR被映射到0x02000000-0x02007FFF
2. 对于非激活的bank,地址会被映射到0x00100000-0x001FFFFF，IFR被映射到0x02008000-0x0200FFFF
3. App在激活的bank中执行，在非激活的bank中升级代码。默认情况下flash0是激活bank
4. 通过IFR中保存的abs版本来决定哪个bank是激活的。abs版本保存在三个不同的IFR phrases中（0th phrase, 1th parase, 511th phrase），使用三次投票机制
5. 当该phrase中高8个byte为0x5a5a5a5a5a5a5a5a,表示低8个byte中保存的abs version是有效的，可以被用来做AB切换比较。
6. 三次投票：
	1. 在初始化阶段，flash硬件将会读取版本号（0 1 511 phrase中的高8个byte和低8个byte共128个bit），如果两个或更多的phrase中的内容是一样的，那么三次投票通过，第八byte的version结果会被输出。
	2. 如果三次投票失败，abs 返回结果为0xFFFFFFFFFFFFFFFF
	3. 当三次投票通过，但是abs version非法（高8个byte非法），输出结果为0xFFFFFFFFFFFFFFFF
7. 初始化阶段，硬件会把flash0 IFR和flash1 IFR中的数据全部读出来，通过对比来获取abs版本号。
	1. 如果一个bank的abs版本三次投票失败，另一个通过，通过的会被设置为激活bank
	2. 两个bank都投票失败，那么flash0会被设置为激活
	3. 两个bank都通过：版本号高的会被设置为激活，如果本版号一样，flash0被设置为激活