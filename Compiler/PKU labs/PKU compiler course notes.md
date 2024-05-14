lexer (extract tokens) => parser (grammar analysis) => AST => IR => assembly code => binary code => executable

example:
program
```c
int main() {
	return 0;
}
```
lexer result:
1. **种类:** 关键字, **内容:** `int`.
2. **种类:** 标识符, **内容:** `main`.
3. **种类:** 其他字符, **内容:** `(`.
4. **种类:** 其他字符, **内容:** `)`.
5. **种类:** 其他字符, **内容:** `{`.
6. **种类:** 关键字, **内容:** `return`.
7. **种类:** 整数字面量, **内容:** `0`.
8. **种类:** 其他字符, **内容:** `;`.
9. **种类:** 其他字符, **内容:** `}`.

parser result:
```c
CompUnit {
  items: [
    FuncDef {
      type: "int",
      name: "main",
      params: [],
      body: Block {
        stmts: [
          Return {
            value: 0
          }
        ]
      }
    }
  ]
}
```

LLVM IR (intermediate representation)
Rust  *\       / x86
C++ -  IR - RISCV
Swift /      \ MIPS
1. **指令选择:** 决定 IR 中的指令应该被翻译为哪些目标指令系统的指令. 例如前文的 Koopa IR 程序中出现的 `lt` 指令可以被翻译为 RISC-V 中的 `slt`/`slti` 指令.
2. **寄存器分配:** 决定 IR 中的值和指令系统中寄存器的对应关系. 例如前文的 Koopa IR 程序中的 `@x`, `%cond`, `%0` 等等, 它们最终可能会被放在 RISC-V 的某些寄存器中. 由于指令系统中寄存器的数量通常是有限的 (RISC-V 中只有 32 个整数通用寄存器, 且它们并不都能用来存放数据), 某些值还可能会被分配在内存中.
3. **指令调度:** 决定 IR 生成的指令序列最终的顺序如何. 我们通常希望编译器能生成一个最优化的指令序列, 它可以最大程度地利用目标平台的微结构特性, 这样生成的程序的性能就会很高. 例如编译器可能会穿插调度访存指令和其他指令, 以求减少访存导致的停顿.