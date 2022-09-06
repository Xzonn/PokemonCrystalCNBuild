## 基本说明
本项目克隆自 CKN&DMG&口袋群星SP 制作的《精灵宝可梦 水晶版》汉化版（[SnDream/pokecrystal_cn_build](https://github.com/SnDream/pokecrystal_cn_build)），目前版本为 V1.2，发布说明[见此链接](https://www.bilibili.com/read/cv17069293)。

该汉化版基于《宝可梦 水晶版》的反编译项目（[pret/pokecrystal](https://github.com/pret/pokecrystal)），后者于 2022 年左右添加了对 3DS Virtual Console 版本添加的补丁的支持，而汉化版尚未对此添加支持，因此本人合并了反编译项目最新代码，以便实现 3DS Virtual Console 版本的连接交换功能。

## 编译
请完全克隆本项目：

``` bash
git clone https://github.com/Xzonn/PokemonCrystalCN.git --recursive
```

然后运行：

``` bash
source env-setup
pmc_finit
pmc_isys
pmc_itext
pmc_build
```

将会在`build`目录下生成`pokecrystal11.gbc`文件，该文件与前述发布说明中的 V1.2 版本 ROM 完全一致，校验码为：

```
MD5: B9BF6AA9905A40AE155BEB7B86AA2CC6
SHA1: 993E46A575007B05407DA74D5FA4307C827750B4
CRC32: 5AD933E5
```

同时生成 Debug 版本的`pokecrystal11_debug.gbc`文件，校验码为：

```
MD5: 927DEDFA8DBEA8E5262D96A725109328
SHA1: 4E326FD85823F55C08E0E28CAC495078F4FA91E2
CRC32: 9716D3A2
```

## Virtual Console 支持
由于本人对 GB 世代的游戏研究有限，虽然已经尽可能实现了代码合并，但仍无法完美编译。目前，在`build`目录下使用`rgbasm vc/pokecrystal11.constants.asm`指令会报错，作为替代方案，可手动将以下内容保存为`build/pokecrystal11.constants.sym`：

```
00:0090 SCREEN_HEIGHT_PX
00:0001 A_BUTTON
00:0016 MAPGROUP_CIANWOOD
00:0008 MAP_CIANWOOD_PHOTO_STUDIO
00:0000 NO_INPUT
00:0002 B_BUTTON
00:0040 D_UP
00:0080 D_DOWN
00:005a FISSURE
00:0078 SELFDESTRUCT
00:0057 THUNDER
00:0094 FLASH
00:0099 EXPLOSION
00:0020 HORN_DRILL
00:003f HYPER_BEAM
00:00d9 PRESENT
00:00d1 anim_1gfx_command
```

然后，在`build`目录下运行：

```
make crystal11_vc
```

将会在生成`pokecrystal11_vc.gbc`文件，校验码为：

```
MD5: 3329256DC77BF98C9404ADF195160BFF
SHA1: 1AD66DF700043A6C4ADDB33BB5192F6EC2F69E56
CRC32: DEFBA27B
```

以及补丁`pokecrystal11.patch`文件，校验码为：

```
MD5: A8D0B8B81A944D5B23EAB31A0955D3EC
SHA1: 68D951E7F503DDEE96D8C7D8C5270382D4D17F69
CRC32: 4F1B02DE
```

运行过程中提示并非所有 ROM 差异均包含于补丁中，目前我尚未发现有效的解决方法。

获得 ROM 和补丁后，可将这两个文件替换 Virtual Console 的 cci 文件的内容，参考代码：

```
3dstool -xvtf cci 0004000000172800.cci --header cciheader.bin -0 0.cxi -1 1.cfa
3dstool -xvtf cxi 0.cxi --header ncchheader.bin --exh exheader.bin --logo logo.bcma.lz --plain plain.bin --exefs exefs.bin --romfs romfs.bin
3dstool -xvtf romfs romfs.bin --romfs-dir romfs
rm romfs/rom/CGBBYTE1.784
cp 0.gbc romfs/rom/CGBBYTE1.784
rm romfs/CGBBYTE1.784.patch
cp 0.patch romfs/CGBBYTE1.784.patch
3dstool -cvtf romfs romfs.bin --romfs-dir romfs
3dstool -cvtf cxi 0.cxi --header ncchheader.bin --exh exheader.bin --logo logo.bcma.lz --plain plain.bin --exefs exefs.bin --romfs romfs.bin
3dstool -cvtf cci 0004000000172800-injected.cci --header cciheader.bin -0 0.cxi -1 1.cfa --not-pad
rm ciheader.bin 0.cxi 1.cfa ncchheader.bin exheader.bin logo.bcma.lz plain.bin exefs.bin romfs.bin
rm romfs -r
```