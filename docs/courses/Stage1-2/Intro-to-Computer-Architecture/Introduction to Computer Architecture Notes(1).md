- <mark>Caller-saved</mark> & <mark>callee-saved</mark> 

**caller-saved** （调用者保护）： <mark>如果一个寄存器在函数调用之后还需要使用原来的值，那么 调用者（ caller ）必须在调用前将它保存到栈上，调用后恢复。</mark> 

**<mark>callee-saved</mark>** <mark>（被调用者保存）</mark> ： <mark>就是上级调用者期望 outer 返回时，在 outer 被修改的 那个 register 保持原值</mark> 

## 1. Commentate: 

Registers are scarce and fast, while stack memory is significantly slower, making the hybrid split necessary to minimize costly data spills and fills across function calls. Caller-saved registers hold short-lived temporaries, requiring the caller to save them only when a live value is needed after a call, whereas Callee-saved registers hold long-lived values and must be preserved by the callee once at function entry and exit. This design confines the save/restore overhead to function boundaries rather than every call site, achieving the optimal trade-off between correctness and runtime performance. 



<!-- Start of picture text -->
2. RISC-V calling convention review (from class)<br>From standard RISC-V conventions:<br>* x1 (ra) - return address. Caller-saved only if the caller makes a function call and needs its own<br>return address later.<br>* x10-x17 (a0-a7) - argument registers, also caller-saved. x10 holds return value from inner.<br>* x5-x7, x28-x31- temporary registers (caller-saved).<br>+ x8-x9, x18-x27 - saved registers (callee-saved). If calle uses them, must save/restore.<br>* x6-x7 - sometimes temps, but in this code, x6 is used before a call, so need to check.<br><!-- End of picture text -->

修改规则 ： 1) X1 ，只要看到 jal ，必存 2) x5-x7 ， x10-x17 ，看调用 inner(after jal code) 后还用不用，用就存 

3) 3)x18-x27 ， x8-x9 ，只要修改必存 



<!-- Start of picture text -->
x1 holds the return address to the caller of outer. The jal instruction used to<br>call the inner procedure overwrites x1 with the inner procedures return address.<br>If x1 is not saved before the call, outer will lose its own return address and<br>will jump to an incorrect location upon completion, causing a program crash or<br>infinite loop.<br><!-- End of picture text -->

(b)step1 stack frame size:RV16 中一个字 =2bytes ，需要保存 n 个寄存器，大小 =2n Step2 offset ：分配 6 字节，新 sp 指向原 sp ， -2n Step3 stack saving ： addi sp, sp, -2n # 分配 6 字节栈空间 

sw x19, 0(sp) # 保存 x19 （偏移 +0 ，占 2 字节） 

Step4 stack release ： lw x19, 0(sp) # 恢复 x19 （先恢复最后存的，镜像对称） 

addi sp, sp, 2n # 释放 6 字节栈空间 

- **Synchronisation and translation** 

Data race: 发生在 lw 和 sw 之间，在一个指令 A 执行 lw( 得到 remain=1) 还没有 sw 之 前，同时指令 B 也得到 remain=1 ， beq 都指向成功导致两指令都以为成功 

- **Write a full assembly code** （ 最后记得写 **jalr x0.x1** ， **0** ）类似的题在 **assignment** 里会有练习到 **Structure:** 

1)selectionsort; 2)outer_loop; 3)inner_loop; 4)no_update; 5)inner_end; 6)outer_end; 7)jarl x0,0(x1) 

#  **Design of Single-Cycle CPU Datapath and Control Unit** 

1) **MemtoReg** <mark>: This signal decides what is written back to the destination register (</mark> rd). If <mark>0, it writes the ALU result back (for R-type/I-type or addi</mark> ).If <mark>1, it writes the data read from Data Memory back (for</mark> lw), If <mark>the</mark> op is sw or <mark>beq they not write register,flled</mark> i by 0 or x 

2) **<mark>MemRead</mark>** <mark>: This enables Data Memory to output the value at the given address.</mark> 1 <mark>reads from memory (only asserted for load instructions like</mark> lw).0 <mark>means no memory read</mark> . 

3) RegWrite: <mark>This enable signal controls whether the register flie is allowed to write data into the</mark> i <mark>destination register (</mark> r <mark>d) on the next clock edge.</mark> 

4) MemWrite <mark>: This enables Data Memory to store data into the given address.</mark> 1 <mark>writes to memory (only asserted for store instructions like</mark> sw).0 <mark>means no memory write.</mark> 

5) Branch <mark>: This indicates the current instruction is a conditional branch (e.g., beq). It does NOT directly change the PC; it is ANDed with the ALU zero fag to generate</mark> l <mark>PCSrc</mark> . <mark>1enables branch evaluation;</mark> 0 <mark>means no branch.</mark> 

ALU input mux ：在来自寄存器的值 _rs2(Read data 2)_ 和来自指令的立即数 _imm_ 二选一 



<!-- Start of picture text -->
+ ORES Ausre = 0 BY, SEETRARMAN (IF Rtype 186, M1 add va, 2, 03) .<br>+ AEH).34 Alusre = 1 BY, iHEMZBDRR (FBT Hype Bf S-type$84, MM addi va, x2, 10 BY Iw / sw ADE<br>ESRD RISC-V SUBIR, ALU APPA:<br>+ AA SRRABERH (rt) .<br>+ BAB RE mux (BD ALU input mux) .<br><!-- End of picture text -->

PC select mux 



<!-- Start of picture text -->
S2RESH PCsre:<br>°. 4 PCsro= Ot, BIE pc + 4 (IFAT)<br>+ BPCsre=1At, BARB tat (SABES)<br><!-- End of picture text -->

就是 beq rs2 ， rs1 ， offset 中 rs1=rs2 时，程序会跳转到目标地址（ PC+ 偏移量）， 而不是顺序执行下一条指令，在硬件上，会使得 PCsrc=1 ，这叫 branch taken 。如 果 rs1 ！ =rs2 ，则程序继续执行下一条指令（ PC+4 ），此时 PCsrc=0, 这叫 branch not taken 

支持的指令 

1 ） R 型： add ， sub ， and ， or 2 ） I 型： lw 3 ） S 型： sw 4 ） B 型： beq ， bne 



<!-- Start of picture text -->
Name 31:25 24:20 19:15 14212 1:7 6:0<br>[Tetype  someciaver 11:0] rst funet3. rd opcode<br>S-type inmed[ 11:5] rs2 rs funct3  inmed[ 4:0] opcode<br>|se-type ismed(s2.10:5} rs2 rst fumet3-_—irmed4:1,12] opcode |<br><!-- End of picture text -->

汇编语法： R 型： op rd, rs1, rs2; I 型： op rd, rs1, imm / op rd, imm(rs1) S 型： op rs2, imm(rs1) B 型： op rs1, rs2, labe1 Jalr rd, rs, offset 

**<mark>ALU</mark>** <mark>控制逻辑</mark> 

### <mark>ALUOp = 00</mark> → <mark>加法（ lw/sw</mark> ） 

<mark>ALUOp = 01</mark> → <mark>减法（ beq</mark> ） <mark>ALUOp = 10</mark> → <mark>根据 funct3</mark> 和 <mark>funct7 决定（</mark> R 型） 



<!-- Start of picture text -->
jEoRe ALUSre MemtoReg Regiirite MemRead Memiirite Branch PcSre (SCRRRER)<br>R-type(e.g. adi) 0 (itrs2) 0 GHALUIER) — 15) ° ° ° 0 (Pc+a)<br>I-type (e.g. addi ) 1 (aeXZB0B) 0 (HALUBR) =—-1(5) ) t) 0) 0 (PC+4)<br>IHtype (e.g. 10) 1 GaRXZBDEX) 1 (EPSFFER) 15) 1 (GPS) ) 0) 0 (Pc+4)<br>S-type (e.g. sw) 1 GaSZBDER) (FeATIB/O) O(RS) 0) 15h) 0) 0 (PC+4)<br>‘SB-type (e.g. beq) 0 (#rs2) (FABIO) O(RS) 0 0 1(BR) SSF branch & Zer<br><!-- End of picture text -->



<!-- Start of picture text -->
add R 0110011 000 0000000<br>sub R 0110011 000 0100000<br>and R 0110011 111 0000000<br>or R 0110011 110 0000000<br>addi I 0010011 000<br>lw I 0000011 010<br>jalr I 1100111 000<br>sw Ss 0100011 010<br>beq SB 1100011 000<br>bne SB 1100011 001<br><!-- End of picture text -->

- 静态链接 **(link time)** ： the administrater must re-link the program against the patched library, adn generating a new executable and replace the old executable compilation or assembly is not need to re-run **<mark>(must re-link,profuce</mark> a** **<mark>new executable code)</mark>** 

<mark>负责解析外部符号引用的翻译层次的哪个阶段？ During the link satge.</mark> The <mark>static linker(ld) use code from static archive and embed</mark> it <mark>directly into fnal</mark> i <mark>executable 动态链接</mark> **<mark>(load time)</mark>** ： <mark>the administrater must</mark> ti <mark>replace the old shared library,patched version</mark> on <mark>disk and restart</mark> it <mark>compilation, assembly</mark> or <mark>linking</mark> is <mark>required</mark> **<mark>(only need</mark> to** **<mark>replace the shared library f</mark> i** **<mark>le,.so,.dil,no re-compilation,no re-assembly,</mark> or** **<mark>re-link)</mark>** <mark>负责解析外部符号引用的翻译层次的哪个阶段？ During load stage.the dynamic linker reaches the symbols when program</mark> is <mark>loaded into memory</mark> 

四 <mark>、五级流水线</mark> 

**IF(instruchtion fetch):** fetches the instruction from memory 

**ID(instruction Decod** e):decodes instruction and reads register values **EX(Execute):** Perform ALU operation or address calculation **MEM(Memory)** :accesses data memory for load/store instruction **WB(Write Back):** Write the result back to the register file 完成一个 clock cycle time 循环的单周期设计，单周期 (single-cycle) 就是将五个流水线的延迟时间加起来。而 pipelined time 流水线设计，是将指令分成 5 个阶段，时钟周期由最慢的阶段决定，执行 n 条指令的时间 = （ n+5-1 ） × 最慢阶段时 间 

- 1. 结构冒险（ **structural hazard** ） ：硬件资源冲突，两条指同时需要同一部件 

### **2.** 数据冒险（ **data hazard** ） ：后续指令需要前一指令的结果，但结果尚未生成 

**RAW(read after write)** ——本质：需要在执行阶段获得新值，如果不处理，会读取旧值，导致错误 

1 ）示例 

add x1, x2, x3   // 指令 A ：将 x2+x3 的结果写入 x1 

sub x4, x1, x5   // 指令 B ：读取 x1 的值，与 x5 做减法 

A 写 X1 ， B 读 x1 

在五级流水线中（ IF,ID,EX,MEM,WB ） : 



<!-- Start of picture text -->
BS eh HA #8 A (add) #8 B (sub)<br>1 IF<br>2 ID IF<br>3 Ex ID<br>4 MEM EX < ieee x1<br>5 WB (5 x1) MEM<br><!-- End of picture text -->

指令 B 在 EX 阶段在 4 周期就要使用 x1 ，但指令 A 的 WB 阶段在 5 周期才将结果写回寄存器，所以在周期 4 时， x1 寄存 器的结果还是旧值，导致 RAW 冒险 

### 2 ）如何判断是否有 RAW 冒险 

取指令 B 的源寄存器编号（ rs1 ， rs2 ），与流水线中前面指令的目的寄存器（ rd ）比较，如果匹配，说明存在 RAW 依赖 lw x1, 0(x2) // 指令 1: 从内存加载到 x1 

add x3, x1, x4 // 指令 2: 使用 x1 

sub x5, x1, x6 // 指令 3: 使用 x1 

or x7, x8, x9 // 指令 4: 与 x1 无关 

这里指令 1 和 2 都存在 RAW 依赖 

3 ）解决方法 

|转发（**Forwarding /**<br>**Bypassing**）|将写指令的结果直接从它所在的流水线阶段（EX/MEM<br>或MEM/WB）送到读指令的ALU输入端，不等写回<br>寄存器|大多数ALU运算指令产<br>生的RAW|
|---|---|---|
|阻塞（**Stall**）|插入一个气泡（bubble），让读指令等待一个或多个周<br>期，直到结果就绪|load-use冒险（load后<br>紧跟使用其结果的指令）|
|编译器调度（**Code**<br>**Scheduling**）|编译器重排指令，插入无关指令或调整顺序，避免<br>RAW冒险|静态调度处理器|



**Forwarding** （转发） 

适用场景： ALU 运算产生的 RAW （就是加减乘除计算的指令，非 load ） 

问题代码： add x1, x2, x3 // 指令 1: x1 = x2 + x3 

sub x4, x1, x5 // 指令 2: x4 = x1 - x5 ( 需要 x1 的最新值 ) 

五级流水线的问题： 

|周期|指令1 (add)|指令2 (sub)|
|---|---|---|
|1|IF||
|2|ID|IF|



|周期|指令1 (add)|指令2 (sub)|
|---|---|---|
|3|EX (计算x1)|ID|
|4|MEM|EX<br>←需要x1|
|5|WB (写x1)|MEM|



添加旁路路径：将指令 1 在 EX 阶段（周期 3 ）计算出的结果直接送到指令 2 的 ALU 输入端 

1 ）周期 3 末：指令 2 的 ALU 结果 =x2+x3 放入 EX/MEM 寄存器 

2 ）周期 4 初：指令 2 进入 EX 阶段，转发单元检测到 RAW ，将 EX/MEM 寄存器的值直接作为 ALU 输入 

3 ）结果：指令 2 得到正确的新值，无需等待 

**Stall** （阻塞） 

适用场景： load-use 冒险（ load 后紧跟使用其结果的指令） 

问题代码： lw x1, 0(x2) // 指令 1: x1 = 从内存加载的值 

add x3, x1, x4 // 指令 2: x3 = x1 + x4 ( 需要 x1 的新值 ) 

Forwarding 不够，因为 load 指令在 MEM 阶段才将数据写入寄存器堆，而 add 指令在 ID/EX 阶段就需要 x1 的值，此时转 发单元无法从 MEM/WB 或 EX/MEM 寄存器获取有效数据；必须插入一个 stall 周期，使 add 指令在 ID 阶段暂停一拍，待 lw 完成 MEM 阶段、结果回写至寄存器堆后，再进入 EX 阶段，从而确保 x1 为最新值。 

周期 **IF ID EX MEM WB** 1 lw 2 add lw (nop ← 检测到 load-use ， add 在 ID 被 stall ， EX 插入气 3 add lw ) 泡 (nop add 4 lw ) (stalled) bubbl 5 bubble lw e bubbl 6 add bubble e 7 add bubble 

周期 3 检测到冒险，将指令 2 的 ID 阶段保持（ stalled ），同时向 EX 阶段注入一个空操作（ nop ） 

周期 4 指令 2 仍为 ID ，周期 5 才进入 EX ，此时指令 1 的 lw 结果已就绪（ forwarding ）代价 ; 损失 1 个时钟周期 

**Code Scheduling** （编译器调度）——换位置 编译器调度后的代码： lw x1, 0(x2) // 加载 **lw x5, 12(x2) //** 把后 适用场景 : 避免 RAW 冒险（尤其是 load-use ），重排指令 面的加载提前，与 **lw x1** 无依赖 问题代码： lw x1, 0(x2) // 加载 add x3, x1, x4 **add x3, x1, x4 //** 使用 **x1** → 必须 // 此时 x1 已就绪（经 **stall 1** 周期 过了一个周期间隔），无需 stall sw x3, 8(x2) // 存储结果 sw x3, 8(x2) lw x5, 12(x2) // 另一个加载 add x6, x5, x7 // x5 add x6, x5, x7 // 使用 x5 → 又一个 也已就绪，无需 stall stall 

## 三种方法总结对比 



<!-- Start of picture text -->
Wik eet BOR BSUMET SSIES set<br>RR at (SHS) a % ({S load-use (38 stall)<br>base at (BRM AT) cy JRE 1 SEBSBB<br>ras SAREE (MAF) a FARMS A (BRTACENITBAN)<br><!-- End of picture text -->

**2.** 控制冒险（ **Control Hazard/Branch Hazard** ）——性能损失 

定义：遇到分支指令（ beq ， bne ）时，下一条指令的地址取决于分支条件是否成立。在流水线中，分支的计算结果往 往要等到 EX 或者 MEM ，而流水线在分支指令之后必须立即取指，否则会空转 

问题代码： 0x00: addi x1, x0, 10 // 无关指令 

0x04: beq x1, x2, TARGET // 如果 x1==x2 则跳转到 TARGET (0x20) 0x08: addi x3, x0, 1 // 分支后面的指令（ fall-through ） 0x0C: sub x4, x5, x6 ... 0x20: TARGET: add x10, x11, x12 

五级流水执行： 

|周期|**IF**|**ID**|**EX**|**MEM**|**WB**|
|---|---|---|---|---|---|
|1|addi|||||
|2|beq|addi||||
|3|addi(0x08)|beq|addi|||
|4|sub(0x0C)|addi(0x08)|beq|addi||
|5|TARGET(0x20)|sub|addi(0x08)|beq|addi|
|6|...|TARGET|sub|addi(0x08)|beq|



在周期 3 的 ID 阶段， beq 比较 x1 ， x2 ，得出相等，同时计算跳转地址，立即修改 PC 

周期 3 的 IF 阶段直接取得 TARGET ，而不是顺序的 addi 

### 局限性—— **locality** （是 **cache** 存在的原理） 

**1.** 时间局限 **temporal locality:** if a memory location is accessed,it is likely to be accessed again soon 例如，循环中的 累加变量。这是缓存 cache 发挥作用的理论基础 

**2.** 空间局限 **spatial locality:** if a memory location is accessed,nearby locations are likely to be 例如，顺序遍历数组 arr[i],arr[i+1]（block size） 

内存层次结构（ memory hierarchy ）——节约时间 

定义：多个不同速度，成本，容量的内存级别组成的系统 

原理：利用局限性，预测接下来会使用哪些内存数据 

Structure （从上到下）：寄存器（ register ），缓存（ SRAM ），主存（ DRAM ），闪存（ FLASH ），磁盘 （ magnetic disk ） 

## **notes** ：每个相邻存储层级存在 **data transfer** ，非相邻不能直接拷贝 **Block** 

内存技术（ **memory technologies** ） 

**1** ． **SRAM RAM:** 靠近处理器的缓存 caches close to the processor 

<mark>盘片（</mark> **<mark>Platter</mark>** <mark>） ：像一张张堆叠的</mark> CD 。 <mark>磁道（</mark> **<mark>Track</mark>** <mark>） ：盘片上的一个个同心圆环（像跑 道）。 扇区（</mark> **<mark>Sector</mark>** ） <mark>：每个磁道切成好多段（像切西 瓜），每段存 512~4096 字节。 读写头（</mark> **<mark>Head</mark>** ） <mark>：像唱针，悬浮在盘片上方几纳 米处，所有磁头连在一起同时移动。</mark> 

结构： 6 个晶体管（ 6T ），比 DRAM 大 6~10 倍、 Traits ： 1 ）一个周期只能读一次或者写一次， 例外：寄存器文件（ register file ）， 2 个读取端口（ rs1 ， rs2 ）， 1 个写端口（ rd ） 

- **2** ） 速度快，不需要刷新，电源不断数据不会丢失 

- **2** ． **DRAM** （动态随机存取存储器） **3** ， **FLASH** （闪存） 

- **4** ． **Magnetic disk** （磁盘） **:** 由三个部分组成（ seek time ， rotational delay ， transfer time ） 

- **1)Seek time:** 磁头 (head) 移动到正确磁道 (track) 所需要的时间 **head is far from desired location** 

- **2** ） **Rotational time** ： 盘片 (platter) 旋转，使所需扇区 (sector) 转到磁头下的时间 **rotation speed is slow** 平均旋转延迟 = 0.5 转 / ( 转速 转 / 秒 ) 

转速 转 / 秒 = 5400 / 60 = 90 转 / 秒 

平均旋转延迟 = 0.5 / 90 = 0.005555... 秒 = 5.56 毫秒 

- **3** ） **Transfer time** ：读取数据的时间，取决于要读多少给字节和磁盘转速 **you need a lot of data** 

**5.** 磁盘和闪存的对比 



<!-- Start of picture text -->
FTE RE (HDD) Af (SSD)<br>z= {RIS (WR) BUR (THAD)<br>BXAS {RE, SCRA Re<br>asa XK (TBR) pS<br>BOBISATE BTS Sets oR TFIRA RABAT BAER ARE ULE)<br>Toy BE HESS, TARA) SEE), yd<br>BUFR Bea. VAISSes TARE, SiCAS<br><!-- End of picture text -->

### 可靠内存（ **Dependable Memory** ） 

定义： <mark>服务实现（ Service Accomplishment</mark> ）， <mark>服务中断（ Service interruption ），间歇性故障（ intermittent failure ），永久性故障（ Permanent failure</mark> ） 

## **Reliability** ：按照预期工作的运行时间比例 

1. MTTF ：能正常工作的平均时长——过于乐观 

2. 2.AFR ：基于 MTTF 计算的一年内发生故障的百分比 

**3.** 公式： AFR = 1 / (MTTF 以年为单位 ) 

### **Availability** ：正常提供服务的时间百分比 

## 1. 公式： 

_MTTF Availability_ = _MTTF_ + _MTTR_ 

Where MTTR=Mean time to repair （减少修复时间和提高 MTTF 同样有效） 

2. 提高 MTTF 的办法 

- 1 ）故障避免 fault avoidance 

- 通过精心构造防止故障发生，例如：高质量组件，设计更好 

- 2 ）故障容错 fault tolerance 

即使一个组件有问题，不影响其他系统，例如：复制，双电源 

- 3 ）故障预测 fault forecasting 

发生前先预测，并且先主动替代，例如： SMART 技术检测 

# **Error Detection** 错误检测 

## 第一种：奇偶校验法 

1. 核心规则 

偶校验 even parity ：所有数据位 + 奇偶位中 **1** 的总个数为偶数 

奇校验 odd parity ：所有数据位 + 奇偶位中 **1** 的总个数为奇数 

2. 具体方法 

例子：偶校验 

Dates ： 0010 1101 1001 0011 

奇偶位 = 0 

- 1 ）计算所有 1 的个数 2 ）如果和是偶数，无错；如果和是奇数，错 

## 3. 理解 



<!-- Start of picture text -->
pease EC ee<br>\‘e . SPEAR) YM > SHEte=0<br>febee fe ee Oea<br>abesas Beos SOBERHUBER) eoNOSENACIS >> SIRE SHBET=0<br><!-- End of picture text -->

## 第二种：汉明错误纠正码 Hamming ECC 

原理：利用多个奇偶位，覆盖不同子集，通常使用偶检验 

规则：奇偶位位于 2 的幂次位置（ 1.2.4.8... ），数据位在其余位置 

|位位<br>置|1|2|3|4|5|6|7|8|9|10|11|1<br>2|
|---|---|---|---|---|---|---|---|---|---|---|---|---|
|编码<br>位|p<br>1|p<br>2|d<br>1|p<br>4|d<br>2|d<br>3|d<br>4|p<br>8|d<br>5|d<br>6|d<br>7|d8|



核心思路： p1 覆盖位置 1,3,5,7 是因为这些位置的二进制最低位是 1 。 

p2 覆盖位置 2,3,6,7 是因为这些位置的二进制第二位是 1 ， p4 覆盖的位置是 4 ， 5 ， 6 ， 7 ， 

示例：存储数据 **1111** 

位置索引； 1(p1),2(p2),3(d1),4(p4),5(d2),6(d3),7(d4) 

先填充数据：位置 : 1 2 3 4 5 6 7 

值 : ? ? 1 ? 1 1 1 

计算 p1 ， p2 ， p3 ： 

p1 负责的位置是 1 ， 3 ， 5 ， 7, 这些位置的已知数据是，位置 3=1. 位置 5=1 ，位置 7=1 ，目前得三个 1 ，是奇数，所以为了 使得 p1 组的总数位偶数， p1 必须 =1 

更新： p1=1 

P2 负责的位置是 2 ， 3 ， 6 ， 7 ，这些位置的已知数据是，位置 3=1 ，位置 6=1 ，位置 7=1 ，目前得到奇数，所以为了使得 p2 组的总位数是偶数， p2 必须 =1 

更新： p2=2 以此类推，得到 p4=1 

最终编码为 1111111 

缺点：无法区分双比特错误和单比特错误，这个问题可以由 **SEC/DED (** 新增一个总校验位 **overall parity, Po)** 方法解决 假设数据翻转了一位，重新计算得到错误的 p2=0 ，将校验组从高位到低位排（ p4,p2,p1 ），结果转十进制就是出错位置 **SEC/DED** 方法 



<!-- Start of picture text -->
TERBSBEHSt (H) SBEMUEHEYE(P.) RH NERS<br>{8 (Even) {% (Even) FAR Siac, IERIMT.<br># (00a) # (oda) Py 12 (SIME) valpaataaiasae.REBUM, NORA,ORR, 5<br>{8 (Even) # (Odd) Sa (P,) BSR. ereEm)ERISA.. (QP, SPS, HIE P, DAT (2x<br>0. 72 REAR, MERRIER,<br>#F (Odd) {% (Even) WAS (RAM) (LHBHAR, HITHIE,<br><!-- End of picture text -->

### 缓存 **cache structure** 

理解：假设主存（ DRAM ）访问需要 100 个时钟周期， Cache 访问只需 1 个周期。如果程序 90% 的访问都能在 Cache 中命 中，则平均访问时间 ≈ 0.9×1 + 0.1×100 = 10.9 周期，远低于 100 周期。 

### **1.** 在引用缓存时，要考虑命中和缺失的问题 

基本操作： 1 ）处理器每次读写数据时，首先检查 cache 

- 2 ） Hit 命中：所需数据在 cache 中，直接使用，不访问主存 

- 3 ） Miss 缺失：所需数据不在 cache 中，必须从主存中读入该数据（以及其相邻的若干字， block ），并存入 

- cache 的一个空槽位，如果 cache 已满 , 则根据替换策略淘汰某个旧块 

**2. miss** 缺失过程中，从主盘重新录入 **cache** 的过程需要消耗时间，称为 **miss penalty** 

_Miss Penalty_ = _Latency_ ¿ _first word_ + _Transfer time for remaining words_ 

Latency to first word 首字延迟， Transfer time 传输时间 

1 ） Instruction cache miss & data cache miss 

Instruction cache miss 在 IF 阶段被触发， PC 将地址发送给主存，导致 fetch stall 。 data cache miss 在 MEM 阶段被触发， 

lw/sw4. 写回主存 

2 ） Write through: 每次写操作均同时更新 Cache 与主存，确保数据一致性，但增加了内存带宽压力。 Write-Back 则仅在 Cache 中标记脏位，仅当块被替换 (dirty bit=1) 时才写回主存，减少了总线流量，提高了性能，但硬件复杂度略高。在发 生 Data Cache Miss 时，若采用 Write-Back 策略，需先检查被替换块是否为脏块，若是则需额外执行一次写回操作，这 将显著增加 Miss Penalty 中的传输时间部分，进而影响整体执行效率。 

### **3.** 内存地址划分 

一个内存地址被分成三个字段（由低到高） 

块内偏移 block offset （ m ） + 索引 index （ n ） + 标签 tag （用于比较是否 hit ， 32-m-n+2 ） 

还存在一个有效位 valid bit （ VB ），如果是 0 ， cache 是空，如果是 1 ， cache 是满 示例： **32** 位地址， **cache** 有 **2**<sup>**n**</sup> 块，每块 **2**<sup>**m**</sup> 字 缓存大小计算公式： 总位数=2 _n ×_ （块大小+ _tag_ 大 +小有效位） 

_tag_ 大小=32 _−_ （ _n_ + _m_ +2） 

E. g:16KiB 的缓存，每块 4 的字，实际需要多少总位？ Solution ： 16 KiB = 16×1024×8 位 = 131072 位（数据存储容量）。 块大小 = 4 字 = 4×32 = 128 位。 所以块数 = 131072/128 = 1024 块 → 2 ¹⁰ = 1024 ，即 n = 10 。 m = log ₂(4) = 2 。 Tag 大小 = 32 − (10 + 2 + 2) = 18 位 每块总位 = 数据 128 + Tag 18 + 有效位 1 = 147 位。 总位数 = 1024×147 = 150528 位 ≈ 18.8 KiB 

4. Associativity 与缺失率 

1 ）直接映射 (Direct-Mapped): 每个内存块只能映射到缓存中的唯一位置，是最简单，但冲突最多的方式 方法： 内存块地址 % 缓存块数 = 缓存索引 2 ）全相联 (Fully Associative): 每个内存块可以放置在 cache 的任何位置，灵活性高，开销大 方法：并行搜索所有缓存块，需要为每个槽位配置一个比较器（ Comparator ） 3 ）组相联 (Set Associative): 每个内存块映射到唯一的一个组，但可以放在组内的任何位置 

关联度 缺失率 1 路（直接映射） 10.3% 2 路组相联 8.6% （提升最大） 4 路组相联 8.3% 8 路组相联 8.1% （收益极小） 

