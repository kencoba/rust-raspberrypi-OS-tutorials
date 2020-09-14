# Before we start

# 始める前に

The following text is a 1:1 copy of the documentation that can be found at the top of the kernel's
main source code file in each tutorial. It describes the general structure of the source code, and
tries to convey the philosophy behind the respective approach. Please read it to make yourself
familiar with what you will encounter during the tutorials. It will help you to navigate the code
better and understand the differences and additions between the separate tutorials.

以下の文書は、各チュートリアルのカーネルのメインソースコードファイルの先頭にある文章のそのままのコピーです。
この文書は、ソースコードの共通構造を記述してあります。また、個々のアプローチの背後にある哲学を伝えようとしています。
これを読んで、チュートリアルの中でどのようなことをするかについて、よく頭に入れてください。
この文書は、コードをよくナビゲートし、個々のチュートリアルごとの違いや追加要素を理解することを助けてくれます。

Please also note that the following text will reference source code files (e.g. `**/memory.rs`) or
functions that won't exist yet in the first bunch of the tutorials. They will be added gradually as
the tutorials advance.

以下のテキストは、チュートリアルの初期段階では存在しないソースコードファイル(例: `**/memory.rs`)や関数を参照することに注意してください。
これらはチュートリアルが進むごとに、徐々に追加されていきます。

Have fun!

では、お楽しみください!

## Code organization and architecture

## コードの構成とアーキテクチャ

The code is divided into different _modules_, each representing a typical **subsystem** of the
`kernel`. Top-level module files of subsystems reside directly in the `src` folder. For example,
`src/memory.rs` contains code that is concerned with all things memory management.

コードは個別の *モジュール*に分割されています。各モジュールは`kernel`の**サブシステム**を表します。
サブシステムのトップレベルにあるモジュールファイルは`src`フォルダ直下にあります。
例えば、`src/memory.rs`はメモリ管理に関する全てのコードを含んでいます。

## Visibility of processor architecture code

## プロセッサ・アーキテクチャ・コードの可視性

Some of the `kernel`'s subsystems depend on low-level code that is specific to the target processor
architecture. For each supported processor architecture, there exists a subfolder in `src/_arch`,
for example, `src/_arch/aarch64`.

`kernel`サブシステムのいくつかは、対象となるプロセッサ・アーキテクチャ特有の、低レベルコードに依存しています。
サポートされているプロセッサ・アーキテクチャごとに、`src/_arch`サブフォルダがあります。
例えば、`src/_arch/aarch64`というフォルダです。

The architecture folders mirror the subsystem modules laid out in `src`. For example, architectural
code that belongs to the `kernel`'s memory subsystem (`src/memory.rs`) would go into
`src/_arch/aarch64/memory.rs`. The latter file is directly included and re-exported in
`src/memory.rs`, so that the architectural code parts are transparent with respect to the code's
module organization. That means a public function `foo()` defined in `src/_arch/aarch64/memory.rs`
would be reachable as `crate::memory::foo()` only.

アーキテクチャのフォルダは、`src`以下にあるサブシステムのモジュールの鏡写しのようになっています。
例えば、`keynel`のメモリサブシステム(`src/memory.rs`)に属するアーキテクチャ・コードは、`src/_arch/aarch64/memory.rs`にあります。
後者のファイルは`src/memory.rs`の中で、直にインクルードされ、再度エクスポートされます。
ですので、アーキテクチャ・コード部は、コードのモジュール編成に対して透過になっています。
これは、`src/_arch/aarch64/memory.rs`の中に定義されたパブリック関数`foo()`は、`crate::memory::foo()`としてしか使えないことを意味します。

The `_` in `_arch` denotes that this folder is not part of the standard module hierarchy. Rather,
it's contents are conditionally pulled into respective files using the `#[path = "_arch/xxx/yyy.rs"]` attribute.

`_arch`の`_`は、このフォルダが標準モジュール階層の一部ではないことを表します。
そうではなくて、このフォルダの内容は`#[path = "_arch/xxx/yyy.rs"]`アトリビュートを使うことで、各々のファイルで状況に応じて呼び出されます。

## BSP code

# BSP コード

`BSP` stands for Board Support Package. `BSP` code is organized under `src/bsp.rs` and contains
target board specific definitions and functions. These are things such as the board's memory map or
instances of drivers for devices that are featured on the respective board.

`BSP`は Board Support Package の略です。
`BSP`コードは、`src/bsp.rs`以下に編成され、対象のボード特有の定義や関数を含んでいます。
例えば、ボードのメモリ・マップや個々のボードに搭載されちているデバイスドライバ・インスタンスがあります。

Just like processor architecture code, the `BSP` code's module structure tries to mirror the
`kernel`'s subsystem modules, but there is no transparent re-exporting this time. That means
whatever is provided must be called starting from the `bsp` namespace, e.g.
`bsp::driver::driver_manager()`.

プロセッサ・アーキテクチャ・コード同様、`BSP`コードのモジュール構造は`kernel`のサブシステム・モジュールの鏡映しになるようにしています。
しかし、これは透過的に再エクスポートできる構造はありません。
提供されるものは全て、`bsp`名前空間で始まる形で呼び出されることになります。
例えば、`bsp::driver::driver_manager()`です。

## Kernel interfaces

## カーネル・インタフェース

Both `arch` and `bsp` contain code that is conditionally compiled depending on the actual target and
board for which the kernel is compiled. For example, the `interrupt controller` hardware of the
`Raspberry Pi 3` and the `Raspberry Pi 4` is different, but we want the rest of the `kernel` code to
play nicely with any of the two without much hassle.

`arch`も`bsp`も、実際のターゲットや、カーネルがコンパイルされるボードに依存して、条件つきでコンパイルされるコードを含んでいます。
例えば、`Raspberry Pi 3`と`Raspberry Pi 4`では、`interrupt controller`ハードウェアは違っています。
ですが私たちとしては、他の部分の`kernel`コードは、この 2 者を気にせずにうまくやっていけるようにしたいのです。

In order to provide a clean abstraction between `arch`, `bsp` and `generic kernel code`, `interface`
traits are provided _whenever possible_ and _where it makes sense_. They are defined in the
respective subsystem module and help to enforce the idiom of _program to an interface, not an
implementation_. For example, there will be a common IRQ handling interface which the two different
interrupt controller `drivers` of both Raspberrys will implement, and only export the interface to
the rest of the `kernel`.

`arch`、`bsp`及び`generic kernel code`の間で綺麗な抽象化を提供するため、_可能な限り_ かつ _合理的な箇所で_、`interface`トレイトが提供されます。
インタフェースは、個々のサブシステム・モジュールの中で定義され、_実装ではなくインタフェースに向かうプログラム_ を実装することを助けます。
例えば、2 種類の Raspberry PI 用に 2 種類の割込みコントローラの`driver`の共通 IRQ ハンドリング・インタフェースが実装されます。
これは`kernel`のその他の部分には、インタフェースのみエクスポートされます。

```
        +-------------------+
        | Interface (Trait) |
        |                   |
        +--+-------------+--+
           ^             ^
           |             |
           |             |
+----------+--+       +--+----------+
| kernel code |       |  bsp code   |
|             |       |  arch code  |
+-------------+       +-------------+
```

# Summary

# まとめ

For a logical `kernel` subsystem, corresponding code can be distributed over several physical
locations. Here is an example for the **memory** subsystem:

論理的な`kernel`サブシステムに関しては、対応するコードは複数の物理的な場所に分散配置できます。
以下は、 **memory** サブシステムの例です。:

- `src/memory.rs` and `src/memory/**/*`
  - Common code that is agnostic of target processor architecture and `BSP` characteristics.
    - Example: A function to zero a chunk of memory.
  - Interfaces for the memory subsystem that are implemented by `arch` or `BSP` code.
    - Example: An `MMU` interface that defines `MMU` function prototypes.
- `src/bsp/__board_name__/memory.rs` and `src/bsp/__board_name__/memory/**/*`
  - `BSP` specific code.
  - Example: The board's memory map (physical addresses of DRAM and MMIO devices).
- `src/_arch/__arch_name__/memory.rs` and `src/_arch/__arch_name__/memory/**/*`

  - Processor architecture specific code.
  - Example: Implementation of the `MMU` interface for the `__arch_name__` processor
    architecture.

- `src/memory.rs` 及び `src/memory/**/*`
  - ターゲット・プロセッサ・アーキテクチャと`BSP`の特性に依存しない、共通コード。
    - 例: メモリチャンクをゼロにする関数。
  - `arch`または`BSP`コードにより実装される、メモリ・サブシステムのためのインタフェース。
    - 例: `MMU`関数プロトタイプを定義する`MMU`インタフェース。
- `src/bsp/__board_name__/memory.rs` 及び `src/bsp/__board_name__/memory/**/*`
  - `BSP`特有のコード。
    - 例:ボードのメモリ・マップ(DRAM や MMIO デバイスの物理アドレス)。
- `src/_arch/__arch_name__/memory.rs` 及び `src/_arch/__arch_name__/memory/**/*`
  - プロセッサ・アーキテクチャ特有のコード。
    - 例: `__arch_name__`プロセッサ・アーキテクチャのための`MMU`インタフェースの実装。

From a namespace perspective, **memory** subsystem code lives in:

名前空間の観点から、 **memory** サブシステムのコードは、以下に配置します。:

- `crate::memory::*`
- `crate::bsp::memory::*`
