<table>
 <tr>
   <td align="center"><img src="https://japan.xilinx.com/content/dam/xilinx/imgs/press/media-kits/corporate/xilinx-logo.png" width="30%"/><h1>2019.1 SDAccel™ 開発環境チュートリアル</h1>
   <a href="https://github.com/Xilinx/SDAccel-Tutorials/branches/all">その他のバージョンを表示</a>
   </td>
 </tr>
 <tr>
 <td align="center"><h1>はじめての SDAccel プログラムの作成</h1>
 </td>
 </tr>
</table>

# 3. アプリケーションのコンパイル、リンク、実行

## 概要

SDAccel アプリケーションは、ソフトウェア プログラムと FPGA バイナリの 2 つのコンポーネントを含むヘテロジニアス アプリケーションです。これらの 2 つのコンポーネントは、専用のコンパイル チェーンを使用してビルドします。この演習では、コマンド ラインから VADD サンプルをコンパイル、リンク、および実行する方法について説明します。

## アプリケーションのビルド

### ホスト プログラムのビルド

OpenCL™ API 呼び出しを使用して C/C++ で記述されたホスト アプリケーションは、標準 CGG コンパイラや XCPP (GCC 周りのシンプルなラッパー) のいずれかを使用してビルドできます。各ソース ファイルはオブジェクト ファイル (o) にコンパイルされ、ザイリンクス SDAccel ランタイム共有ライブラリとリンクされて、実行ファイル (EXE) が作成されます。GCC および関連するコマンド ライン オプションの詳細は、[GNU コンパイラ コレクション (GCC) の使用](https://gcc.gnu.org/onlinedocs/gcc/)を参照してください。

ホスト コードをビルドするには、`./reference-files/src` ファイルに次の XCPP コマンドを使用します。

```bash
xcpp -I$XILINX_XRT/include/ -I$XILINX_VIVADO/include/ -Wall -O0 -g -std=c++14 ./src/host.cpp  -o 'host'  -L$XILINX_XRT/lib/ -lOpenCL -lpthread -lrt -lstdc++
```

このコマンドはザイリンクス ランタイム (XRT)、Vivado ツール、OpenCL API に必要なライブラリおよびインクルード ファイルを指定します。ホストのビルド方法の詳細は、「アクセラレーションされたアプリケーションをビルドして実行するための基本的な概念」チュートリアルの[アプリケーションのビルド](../Pathway3/BuildingAnApplication.md)演習を参照してください。

> **ヒント**: ここに示すコマンドを実行するには、「アプリケーションのビルド」で説明するように、SDx 開発ツールおよび XRT 用の環境を設定する必要があります。

### FPGA バイナリのビルド

ソフトウェア アプリケーションと同様、ハードウェアのビルドには、XOCC コンパイラを使用して [コンパイル](../Pathway3/BuildingAnApplication.md#hardware-compilation)および[リンク](../Pathway3/BuildingAnApplication.md#hardware-linking)をする必要があります。

次のコマンドの場合、ビルド ターゲットはソフトウェア エミュレーション (`-t sw_emu`) です。「アクセラレーションされたアプリケーションをビルドして実行するための基本的な概念」チュートリアルの[アプリケーションのビルド](../Pathway3/BuildingAnApplication.md)演習で説明したように、`hw_emu` および `hw` などの別のビルド ターゲットを指定することもできます。

* 次のコマンドを使用して VADD サンプルのハードウェア カーネルをコンパイルします。

  ```bash
  xocc -t sw_emu --platform xilinx_u200_xdma_201830_1 -g -c -k vadd -I'../src' -o'vadd.xilinx_u200_xdma_201830_1.xo' './src/vadd.cpp'
  ```

* 次のコマンドを使用して VADD サンプルのハードウェア カーネルをリンクします。

  ```bash
  xocc -t sw_emu --platform xilinx_u200_xdma_201830_1 -g -l --nk vadd:1:vadd_1 -o'vadd.xilinx_u200_xdma_201830_1.xclbin' vadd.xilinx_u200_xdma_201830_1.xo
  ```

## エミュレーションの実行

ハードウェアでデザインを実行する前に、[ソフトウェアおよびハードウェア エミュレーションの実行](../Pathway3/Emulation.md)モードでデザインを実行して、機能を検証しておいてください。どのエミュレーション モードでもアプリケーションを実行するのは、次の 2 つのプロセスです。

1. エミュレーション コンフィギュレーション ファイルを生成。
2. アプリケーションを実行。

### ソフトウェア エミュレーション

1. ソフトウェア エミュレーション モード (`-t sw_emu`) で実行するには、エミュレーション コンフィギュレーション ファイルを生成し、XCL\_EMULATION\_MODE 環境変数を設定します。

   ```bash
   emconfigutil --platform xilinx_u200_xdma_201830_1
   export XCL_EMULATION_MODE=sw_emu  
   ```

2. コンフィギュレーション ファイル (`emconfig.json`) を生成して XCL\_EMULATION\_MODE を設定したら、次のコマンドでホスト プログラムとカーネルをソフトウェア エミュレーション モードで実行します。

   ```bash
   ./host vadd.xilinx_u200_xdma_201830_1.xclbin
   ```

3. エミュレーション実行を確認します。

   ```
   Found Platform
   Platform Name: Xilinx
   INFO: Reading vadd.xilinx_u200_xdma_201830_1.xclbin
   Loading: 'vadd.xilinx_u200_xdma_201830_1.xclbin'
   TEST PASSED
   ```

> **重要:** RTL カーネルでは、C モデルが関連付けられている場合にソフトウェア エミュレーションがサポートされます。C モデルが使用可能でない場合は、ハードウェア エミュレーションを使用してカーネル コードをデバッグする必要があります。

### ハードウェア エミュレーション

1. ハードウェア エミュレーションの場合は、`-t hw_emu` XOCC オプションを使用してカーネルをビルドし直します。

   ```bash
   xocc -t hw_emu --platform xilinx_u200_xdma_201830_1 -g -c -k vadd -I'../src' -o'vadd.xilinx_u200_xdma_201830_1.xo' './src/vadd.cpp'
   xocc -t hw_emu --platform xilinx_u200_xdma_201830_1 -g -l --nk vadd:1:vadd_1 -o'vadd.xilinx_u200_xdma_201830_1.xclbin' vadd.xilinx_u200_xdma_201830_1.xo
   ```

2. ハードウェア エミュレーション モードで実行するには、エミュレーション コンフィギュレーション ファイルを生成し、XCL\_EMULATION\_MODE 環境変数を設定します。

   ```bash
   emconfigutil --platform xilinx_u200_xdma_201830_1
   export XCL_EMULATION_MODE=hw_emu
   ```

3. 次のコマンドを使用し、ホスト プログラムとカーネルをハードウェア エミュレーション モードで実行します。

   ```
   ./host vadd.xilinx_u200_xdma_201830_1.xclbin
   ```

4. エミュレーション実行を確認します。

   ```
   Found Platform
   Platform Name: Xilinx
   INFO: Reading vadd.xilinx_u200_xdma_201830_1.xclbin
   Loading: 'vadd.xilinx_u200_xdma_201830_1.xclbin'
   INFO: [HW-EM 01] Hardware emulation runs simulation underneath. Using a large data set will result in long simulation times. It is recommended that a small dataset is used for faster execution. The flow uses approximate models for DDR memory and interconnect and hence the performance data generated is approximate.
   TEST PASSED
   INFO: [SDx-EM 22] [Wall clock time: 16:39, Emulation time: 0.108767 ms] Data transfer between kernel(s) and global memory(s)
   vadd_1:m_axi_gmem-DDR[1]          RD = 32.000 KB              WR = 16.000 KB
   ```

## ハードウェアでの実行

エミュレーション モードを使用して機能の正しさとホストとカーネルの統合を検証したら、次はザイリンクス FPGA の Alveo アクセラレータ カードをターゲットにします (`-t hw`)。

### ハードウェア実行

1. ハードウェアの場合は、`-t hw` XOCC オプションを使用してカーネルをビルドし直します。

   ```bash
   xocc -t hw_emu --platform xilinx_u200_xdma_201830_1 -g -c -k vadd -I'../src' -o'vadd.xilinx_u200_xdma_201830_1.xo' './src/vadd.cpp'
   xocc -t hw_emu --platform xilinx_u200_xdma_201830_1 -g -l --nk vadd:1:vadd_1 -o'vadd.xilinx_u200_xdma_201830_1.xclbin' vadd.xilinx_u200_xdma_201830_1.xo
   ```

2. XCL\_EMULATION\_MODE 環境は設定しないようにします。

   ```
   unset XCL_EMULATION_MODE
   ```

   この段階ではデザインを Vivado ツールで合成して FPGA ハードウェアに読み込むビットストリームにパッケージする必要があるので、ハードウェア ビルドに時間がかかる可能性があります。

3. ビルドが準備でき、環境変数を正しく設定したら、デザインを Alveo アクセラレータ カードで実行します。

   ```
   ./host vadd.xilinx_u200_xdma_201830_1.xclbin
   ```

4. エミュレーション実行を確認します。

   ```
   Found Platform
   Platform Name: Xilinx
   INFO: Reading vadd.xilinx_u200_xdma_201830_1.xclbin
   Loading: 'vadd.xilinx_u200_xdma_201830_1.xclbin'
   TEST PASSED
   ```

## makefile の使用

makefile は、コマンドおよびオプションを整理する単純な方法で、必要に応じてビルドを繰り返しやすくします。`./reference-files` に含まれる `Makefile` には、上記で定義したコマンドが含まれており、ビルドをクリーンアップし、プロジェクトをビルドし直すための構造も含まれます。

この `Makefile` を使用するには、使用しているホスト コードのビルド ターゲット (TARGET)、ハードウェア (DEVICE)、およびバージョン (VER) を指定する必要があります。

* ソフトウェア エミュレーションをターゲットにする場合は、次の `make` コマンドを使用します。

  ```
    make check TARGET=sw_emu DEVICE=<DEVICE> VER=host_cpp
  ```

* ハードウェア エミュレーションをターゲットにする場合は、次の `make` コマンドを使用します。

  ```
  make check TARGET=hw_emu DEVICE=<DEVICE> VER=host_cpp
  ```

* ハードウェアをターゲットにするには、次の `make` コマンドを使用します。

  ```
  make check TARGET=hw DEVICE=<DEVICE> VER=host_cpp
  ```

## 次のステップ

次のステップは、[アプリケーションのプロファイル](./profile_debug.md)です。

</br>
<hr/>
<p align= center><b><a href="../../README.md">メイン ページに戻る</a> — <a href="../sdaccel-getting-started/README.md">入門コースの初めに戻る</a></b></p>
<p align="center"><sup>Copyright&copy; 2019 Xilinx</sup></p>
