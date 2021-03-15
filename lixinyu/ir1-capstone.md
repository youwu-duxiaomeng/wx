# x86-qemu-mips ir1/capstone documentation



由于distrom支持的指令集不够完善，且很少(几乎没有)更新，其支持支持指令集最该为avx(sandy bridge 开始支持)，不支持avx2(haswell开始支持)。所以要将distorm替换为capstone，capstone也是qemu的一部分，使用起来方便。



## 0x0 IR1_INST结构 

注意：除了 IR1_INST外所有的结构信息都位于capstone.h和x86.h中

###  IR1_INST：

```c
typedef struct IR1_INST {
    cs_insn *info;                       //capstone 的指令结构体指针
    uint8_t _eflag_use;           //该指令需要使用的eflags
    uint8_t _eflag_def;           //该指令需要计算的的eflags，
    uint8_t _native_inst_num;  //该指令对应的mips指令条数
    uint8_t flags;                       //该指令的一些flags，可以再ir1.h中查看，作用不太大
} IR1_INST;
```



### cs_insn ：指令的详细信息

由于capstone是个多平台的反汇编引擎，其中有不少的冗余信息

注意：capstone提供了一种简便模式，会有些信息不会产生，以加快速度，但是我们不使用这个模式，所以所有的可能会产生的信息都会产生。

```c
struct cs_insn {
	/// 指令ID(基本上是一个用于指令助记符的数字ID)
	/// 应在相应架构的头文件中查找'[ARCH]_insn' enum中的指令id，如ARM.h中的'arm_insn'代表ARM, X86.h中的'x86_insn'代表X86等…
	unsigned int id;

	/// 指令地址 (EIP)
	uint64_t address;

	/// 指令长度（byte）
	uint16_t size;

	/// 此指令的机器码，其字节数由上面的@size表示
	uint8_t bytes[16];

	/// 指令的Ascii文本助记符，opcode
	char mnemonic[CS_MNEMONIC_SIZE];

	/// 指令操作数的Ascii文本
	char op_str[160];

	/// cs_detail指针
	cs_detail *detail;
} cs_insn;
```



### cs_detail

```c
struct cs_detail {
	uint16_t regs_read[12]; ///< 这个参数读取隐式寄存器列表
	uint8_t regs_read_count; ///< 这个参数读取隐式寄存器计数

	uint16_t regs_write[20]; ///< 这个参数修改隐式寄存器列表
	uint8_t regs_write_count; ///< 这个参数修改隐式寄存器计数

	uint8_t groups[8]; ///< 此指令所属的指令组的列表，可以在x86.h:x86_insn_group查看
	uint8_t groups_count; ///< 此指令所属的组的数

	/// 特定于体系结构的信息
	union {
		cs_x86 x86;     ///< X86 架构, 包括 16-bit, 32-bit & 64-bit 模式，我们只需要这一个
		cs_arm64 arm64; ///< ARM64 架构 (aka AArch64)
		cs_arm arm;     ///< ARM 架构 (包括 Thumb/Thumb2)
		cs_m68k m68k;   ///< M68K 架构
		cs_mips mips;   ///< MIPS 架构
		cs_ppc ppc;	    ///< PowerPC 架构
		cs_sparc sparc; ///< Sparc 架构
		cs_sysz sysz;   ///< SystemZ 架构
		cs_xcore xcore; ///< XCore 架构
		cs_tms320c64x tms320c64x;  ///< TMS320C64x 架构
		cs_m680x m680x; ///< M680X 架构
		cs_evm evm;	    ///< Ethereum 架构
	};
} cs_detail;
```



### cs_x86：capstone中x86指令的详细信息

```c
typedef struct cs_x86 {
	///指令前缀，最多占4字节
	///0表示没有该前缀
	/// prefix[0] ： REP/REPNE/LOCK
	/// prefix[1] ：段重写 (x86_64时为0):
	/// prefix[2] ：操作数大小重写
	/// prefix[3] ：地址操作数大小重写
	uint8_t prefix[4];

	/// 指令的opcode，
	/// VEX指令前缀视为opcode的一部分
	uint8_t opcode[4];

	/// REX prefix: only a non-zero value is relevant for x86_64
	uint8_t rex;

	/// Address size, which can be overridden with above prefix[5].
	uint8_t addr_size;

	/// ModR/M byte
	uint8_t modrm;

	/// SIB value, or 0 when irrelevant.sib的信息会直接体现在mem_opnd内，一般不用考虑
	uint8_t sib;

	/// Displacement value, valid if encoding.disp_offset != 0
	int64_t disp;

	/// SIB index register, or X86_REG_INVALID when irrelevant.
	x86_reg sib_index;
	/// SIB scale, only applicable if sib_index is valid.
	int8_t sib_scale;
	/// SIB base register, or X86_REG_INVALID when irrelevant.
	x86_reg sib_base;

	/// XOP Code Condition
	x86_xop_cc xop_cc;

	/// SSE Code Condition
	x86_sse_cc sse_cc;

	/// AVX Code Condition
	x86_avx_cc avx_cc;

	/// AVX Suppress all Exception
	bool avx_sae;

	/// AVX static rounding mode
	x86_avx_rm avx_rm;

	
	union {
        //该指令对eflag的使用情况，由于不是非常正确，现在并未使用
		/// EFLAGS updated by this instruction.
		/// This can be formed from OR combination of X86_EFLAGS_* symbols in x86.h
		uint64_t eflags;
		/// FPU_FLAGS updated by this instruction.
		/// This can be formed from OR combination of X86_FPU_FLAGS_* symbols in x86.h
		uint64_t fpu_flags;
	};

	/// 操作数数目
	uint8_t op_count;

	cs_x86_op operands[8];	///操作数

	cs_x86_encoding encoding;  ///< encoding information
} cs_x86;
```



### cs_x86_op：x86操作数

```c
typedef struct cs_x86_op {
		x86_op_type type;	///< operand type，X86_OP_REG,X86_OP_IMM,X86_OP_MEM
		union {
			x86_reg reg;	  ///< register value for REG operand
			int64_t imm;		///< immediate value for IMM operand,使用时请结合下面的size，以防出错
			x86_op_mem mem;		///< base/index/scale/disp value for MEM operand
		};

		/// size of this operand (in bytes).
		uint8_t size;

		/// How is this operand accessed? (READ, WRITE or READ|WRITE)
		/// This field is combined of cs_ac_type.
		/// NOTE: this field is irrelevant if engine is compiled in DIET mode.
		uint8_t access;//访问情况，读、写、读写

		/// AVX broadcast type, or 0 if irrelevant
		x86_avx_bcast avx_bcast;

		/// AVX zero opmask {z}
		bool avx_zero_opmask;
} cs_x86_op;
```



## 0x1 API

csh：用于生成调用capstone API的句柄 `size_t csh` ，其实就是用来保存包括反汇编架构，一些设置等，cs_open获取，以后每次调用api都要带上，让capstone知道当前的设置。

```c
/**
 Initialize CS handle: this must be done before any usage of CS.

 @arch: architecture type (CS_ARCH_*)
 @mode: hardware mode. This is combined of CS_MODE_*
 @handle: pointer to handle, which will be updated at return time

 @return CS_ERR_OK on success, or other value on failure (refer to cs_err enum
 for detailed error).
*/
CAPSTONE_EXPORT
cs_err CAPSTONE_API cs_open(cs_arch arch, cs_mode mode, csh *handle);


实际使用,ir1_init:
cs_open(CS_ARCH_X86, CS_MODE_32, &handle)
```



### cs_disasm：核心api

```c
 @handle: handle returned by cs_open()
 @code: buffer containing raw binary code to be disassembled.
 @code_size: size of the above code buffer.
 @address: address of the first instruction in given raw code buffer.
 @insn: array of instructions filled in by this API.
	   NOTE: @insn will be allocated by this function, and should be freed
	   with cs_free() API.
 @count: number of instructions to be disassembled, or 0 to get all of them

 @return: the number of successfully disassembled instructions,
 or 0 if this function failed to disassemble the given code

 On failure, call cs_errno() for error code.
*/
CAPSTONE_EXPORT
size_t CAPSTONE_API cs_disasm(csh handle,
		const uint8_t *code, size_t code_size,
		uint64_t address,
		size_t count,
		cs_insn **insn);
//注意:内存由capstone进行分配
实际使用, ir1_disasm:
int count = cs_disasm(handle, addr, 15, (uint64_t)t_pc, 1, &info);
```



其它API：

cs_reg_name ：根据reg获取reg的ascii名称

cs_insn_name ：根据opcode获取指令的ascii名称

cs_malloc、cs_free:分配、释放cs_insn

还有一些api暂未使用，如果需要可以直接看capstone.h



## 0x2 替换distorm的过程

1、替换IR1_INST的表示，然后替换返汇编函数，这样IR1_INST就是完整的具有所需要的信息

2、替换ir1.h/ir1.c的各个结构体，直接使用typedef，减少代码修改处

```c
typedef x86_insn IR1_OPCODE;
typedef x86_op_type IR1_OPND_TYPE;
typedef cs_x86_op IR1_OPND;
typedef x86_prefix IR1_PREFIX;
typedef x86_insn_group IR1_OPCODE_TYPE;
```

2、替换ir1.h/ir1.c的各个函数接口，要保持语义不变，尽量和原函数功能一样，谢姐改动有

- capstone的x86_op_mem，包含段寄存器，重点在于串指令。但是原IR1认为只要是使用默认段寄存器的都是没有段寄存器。所以添加验证：没有段寄存器重写前缀的都使其op_mem中的段寄存器失效（相当于无段寄存器），以保持语义一致。
- 指令opcode匹配，capstone的opcode要比原IR1_INST多了不少，而且opcode名称也不一定相同。先将相同的进行用程序进行替换，不同的手动替换，不同的主要有
  - 1jz,setz等变成了je,sete
  - movs cmos scas lods stos这些串指令在capstone中多了后缀movsb，movsw，movsd，movsq，所以在遇到这些指令时进行适当修改。
  - movsd（movs）和movsd（sse指令）居然采用的是同一个id，所以一些情况下遇到时需要根据opcode来判断。
  - 原IR1_INST有num来表示是第几个寄存器，capstone只有寄存器名字，需要根据寄存器名字来获得num
  - 如果是分支指令，其中的立即数，在capstone中是绝对地址，而IR1_INST中是相对地址，需要修改。
  - capstone中没有callin,jmpin这种指令，用到时需要做处理。
  - 部分eflags不一致，最初是按opcode进行修改，最后直接采用原来的比较方便，也无错误。
  - 部分指令操作数顺序不一样，如stos，两个操作数位置不一样，很长时间才找出错误
  - 部分fpu指令操作数个数不同，含义不同，需要修改
  - 等等一些细节地方
- 将直接对IR1_INST成员进行引用的地方修改
- 大量修改tr_opnd_process.c中的操作数问题，由于capstone和IR1_INST的操作数类型不同
- 等等还有不少细节

## 0x3 遗留问题

- capstone结构体较大大概一条指令有2000字节左右，比较占用内存。现在没有释放的操作，应该调用cs_free予以释放。
- 部分未使用的函数和部分未遇到的情况直接采用assert(0)来处理，由于即使实现了也无法验证正确性，以后遇到时可能很难排查错误，所以不予以实现，用到时候再予以实现。

## 0x4 其他说明

- 我们需要的capstone的信息都在capstone.h和x86.h中，这两个文件中都有非常详细的说明，qemu/capstone/include/capstone/capstone.h..x86.h
- 还有两个文件qemu/capstone/cstool/cstool_x86.c..cstool.c，这两个文件内有不少关于结构体信息如何使用的示例。
- 在capstone目录下(qemu自带的或者git clone的都行)./make.sh  && ./make.sh install 即可安装capstone库，这样系统中会有个cstool(就是上面cstool.c 编译后的结果)，这个可以输出指令的详细信息，像下面这样，遇到指令不确定结果的时候，用这个工具很方便。

```bash
sample » cstool -d x32 8d4dfc 0x804881f                                                                                                ~/m/sample
804881f  8d 4d fc                                         lea	ecx, [ebp - 4]
	Prefix:0x00 0x00 0x00 0x00 
	Opcode:0x8d 0x00 0x00 0x00 
	rex: 0x0
	addr_size: 4
	modrm: 0x4d
	disp: 0xfffffffffffffffc
	sib: 0x0
	op_count: 2
		operands[0].type: REG = ecx
		operands[0].size: 4
		operands[0].access: WRITE
		operands[1].type: MEM
			operands[1].mem.base: REG = ebp
			operands[1].mem.disp: 0xfffffffffffffffc
		operands[1].size: 4
		operands[1].access: READ
	Registers read: ebp
	Registers modified: ecx
	Groups: not64bitmode 
```

