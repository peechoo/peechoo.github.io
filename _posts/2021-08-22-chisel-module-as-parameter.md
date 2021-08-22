---
title: "[Chisel] 将 Module 作为类参数"
date: 2021-08-22
author: zyk
---

在 Chisel 里使用 `Module` 作为生成另一个 `Module` 的参数。

## 代码示例

```scala
import chisel3.RawModule
import chisel3.experimental.BaseModule
import chisel3.stage.ChiselStage

// Provides a more specific interface since generic Module
// provides no compile-time information on generic module's IOs.
trait MyAdder {
    def in1: UInt
    def in2: UInt
    def out: UInt
}

class Mod1 extends RawModule with MyAdder {
    val in1 = IO(Input(UInt(8.W)))
    val in2 = IO(Input(UInt(8.W)))
    val out = IO(Output(UInt(8.W)))
    out := in1 + in2
}

class Mod2 extends RawModule with MyAdder {
    val in1 = IO(Input(UInt(8.W)))
    val in2 = IO(Input(UInt(8.W)))
    val out = IO(Output(UInt(8.W)))
    out := in1 - in2
}

class X[T <: BaseModule with MyAdder](genT: => T) extends Module {
    val io = IO(new Bundle {
        val in1 = Input(UInt(8.W))
        val in2 = Input(UInt(8.W))
        val out = Output(UInt(8.W))
    })
    val subMod = Module(genT)
    io.out := subMod.out
    subMod.in1 := io.in1
    subMod.in2 := io.in2
}

println(ChiselStage.emitVerilog(new X(new Mod1)))
println(ChiselStage.emitVerilog(new X(new Mod2)))
```

分析：

- `MyAdder` 可以理解为 `Mod1` 和 `Mod2` 两个 `Module` 的接口定义。
- `Mod1` 和 `Mod2` 接口相同，功能不同。
- `X` 接收一个实现了 `MyAdder` 特质的对象作为参数。
- `: =>` 表示参数会延迟求值。

### Verilog 代码生成

1. `genT = Mod1`

```verilog
module Mod1(
  input  [7:0] in1,
  input  [7:0] in2,
  output [7:0] out
);
  assign out = in1 + in2; // @[polymorphism-and-parameterization.md 174:16]
endmodule
module X(
  input        clock,
  input        reset,
  input  [7:0] io_in1,
  input  [7:0] io_in2,
  output [7:0] io_out
);
  wire [7:0] subMod_in1; // @[polymorphism-and-parameterization.md 192:24]
  wire [7:0] subMod_in2; // @[polymorphism-and-parameterization.md 192:24]
  wire [7:0] subMod_out; // @[polymorphism-and-parameterization.md 192:24]
  Mod1 subMod ( // @[polymorphism-and-parameterization.md 192:24]
    .in1(subMod_in1),
    .in2(subMod_in2),
    .out(subMod_out)
  );
  assign io_out = subMod_out; // @[polymorphism-and-parameterization.md 193:12]
  assign subMod_in1 = io_in1; // @[polymorphism-and-parameterization.md 194:16]
  assign subMod_in2 = io_in2; // @[polymorphism-and-parameterization.md 195:16]
endmodule
```

2. `genT = Mod2`

```verilog
module Mod2(
  input  [7:0] in1,
  input  [7:0] in2,
  output [7:0] out
);
  assign out = in1 - in2; // @[polymorphism-and-parameterization.md 182:16]
endmodule
module X(
  input        clock,
  input        reset,
  input  [7:0] io_in1,
  input  [7:0] io_in2,
  output [7:0] io_out
);
  wire [7:0] subMod_in1; // @[polymorphism-and-parameterization.md 192:24]
  wire [7:0] subMod_in2; // @[polymorphism-and-parameterization.md 192:24]
  wire [7:0] subMod_out; // @[polymorphism-and-parameterization.md 192:24]
  Mod2 subMod ( // @[polymorphism-and-parameterization.md 192:24]
    .in1(subMod_in1),
    .in2(subMod_in2),
    .out(subMod_out)
  );
  assign io_out = subMod_out; // @[polymorphism-and-parameterization.md 193:12]
  assign subMod_in1 = io_in1; // @[polymorphism-and-parameterization.md 194:16]
  assign subMod_in2 = io_in2; // @[polymorphism-and-parameterization.md 195:16]
endmodule
```

Verilog 代码的逻辑与预期一致。

## 其它细节

- `RawModule` / `BaseModule` / `Module` 中：`RawModule` 没有隐式 `clock` 和 `reset` 端口；`BaseModule` 是其它 `Module` 的抽象基类（Abstract base class）；`Module` 是定义一个模块时通常会使用的类。
- `MultiIOModule` 比起 `RawModule` 多了隐式 `clock` 和 `reset` 端口。

## 参考资料

- Chisel/FIRRTL Doc: [Parametrization based on Modules](https://www.chisel-lang.org/chisel3/docs/explanations/polymorphism-and-parameterization.html#parametrization-based-on-modules)
- Module.scala: https://github.com/chipsalliance/chisel3/blob/v3.4.3/core/src/main/scala/chisel3/Module.scala
- RawModule.scala: https://github.com/chipsalliance/chisel3/blob/v3.4.3/core/src/main/scala/chisel3/RawModule.scala