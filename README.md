# 烧录 WiFiDuck 到 CJMCU-3212 上的一种方法

该方法参考 [puckk/CJMCU-3212](https://github.com/puckk/CJMCU-3212)（其方法可以烧录旧版的 [wifi_ducky](https://github.com/spacehuhn/wifi_ducky)），感谢 puckk！

请注意：

* 此方法基于 Windows 操作系统。
* 我不是专家，只是希望帮助一些毫无头绪的人。
* 这不是一个最佳方案，过程中很多选项都可以被替换，这只是使其能够工作的一种方法。
* 此方法无法使用 SD 卡，并且 LED 灯（13）不工作。
* 请按照顺序操作，否则可能会失败。

## 1. 检查串行设备参数（如果不能烧录成功）

进入 Windows `设备管理器`，展开 `端口（COM 和 LPT）` 类目，右键 CJMCU-3212 对应的的 `USB 串行设备（COM*）`，点击 `属性` 打开属性窗口，点击选择第二个选项卡 `端口设置`，并按如下表格设置后点击确定关闭窗口。（下面是我使用的参数，参数不一定最合适，但有效。因此参数值应该不一定需要一致，我将其全部写出只是为了避免其他未知问题，后续表格同理）

| 参数     | 选项 |
| -------- | ---- |
| 位/秒    | 9600 |
| 数据位   | 8    |
| 奇偶校验 | 无   |
| 停止位   | 1    |
| 流控制   | 无   |

## 2. 烧录来自 puckk 的 [step1.ino](https://github.com/puckk/CJMCU-3212/blob/master/step1.ino) 文件

启动 [Arduino IDE](https://www.arduino.cc/en/main/software)，点击顶部 `工具` 菜单，选择第三个集合中第一项 `开发板: "*"`，选择 `Arduino Leonardo`；选择第三个集合中第二项 `端口`，选择 `设备管理器` 中显示的端口（每次烧录都要确保端口正确，下文不再重复）。

复制以下程序到编辑窗口，点击顶部 `项目` 菜单，选择 `上传` 选项，烧录程序。

```c++
int program_pin = 12;
int enable_pin = 13;

void setup()
{
  Serial1.begin(115200);
  Serial.begin(115200);
  pinMode(enable_pin, OUTPUT);
  pinMode(program_pin, OUTPUT);
  digitalWrite(program_pin, LOW);
  digitalWrite(enable_pin,HIGH);
}

void loop()
{
  while(Serial1.available()){
    Serial.write((uint8_t)Serial1.read());
  }

  if(Serial.available()){
    while(Serial.available()){
      Serial1.write((uint8_t)Serial.read());
    }
  }
}
```

## 3. 添加 WiFi Duck 开发板到 Arduino

启动 Arduino IDE，点击顶部 `文件` 菜单，选择倒数第二项 `首选项`，在开启的对话框中找到第一个选项卡（`设置`）中的 `附加开发板管理器网址:`，点击输入框右侧按钮并在新对话框中的编辑框中的新的一行输入 `https://raw.githubusercontent.com/spacehuhn/hardware/master/wifiduck/package_wifiduck_index.json`，依次点击两个窗口底部的 `好` 按钮完成设置。

点击顶部 `工具` 菜单，选择第三个集合中第一项 `开发板: "*"`，选择第一项 `开发板管理器...`，在打开的对话框中搜索 `wifi duck`，然后在结果中分别安装 `WiFi Duck ESP8266 Boards` 和 `WiFi Duck AVR Boards`（将鼠标移至上方将显示 `安装` 按钮。根据网络状况速度可能会较慢，在中国可能甚至无法下载），然后关闭窗口。

## 4. 烧录 ESP8266 程序

下载 Zip 包或克隆 [SpacehuhnTech/WiFiDuck](https://github.com/SpacehuhnTech/WiFiDuck) 仓库，（解压后）使用 Arduino IDE 打开 `esp_duck/esp_duck.ino` 文件。

点击 Arduino IDE 顶部 `工具` 菜单，选择第三个集合中第一项 `开发板: "*"`，选择第二个集合中的 `WiFi Duck ESP8266`，再选择 `Generic ESP8266 Module`。

再次点击 `工具` 菜单将出现很多可选参数，请依照下表设置：

| 参数              | 选项                              |
| ----------------- | --------------------------------- |
| Debug             | Disabled                          |
| Protocol          | Serial                            |
| Upload Speed      | 115200                            |
| CPU Frequency     | 80 MHz                            |
| Crystal Frequency | 26 MHz                            |
| Flash Size        | 4MB (FS:3MB OTA:~512KB)           |
| Flash Mode        | DIO                               |
| Flash Frequency   | 40 MHz                            |
| Reset Method      | dtr (aka nodemcu)                 |
| Debug port        | Disabled                          |
| Debug Level       | 无                                |
| lwIP Variant      | v2 Lower Memory                   |
| VTables           | Flash                             |
| Exceptions        | Legacy (new can return nullptr)   |
| Erase Flash       | All Flash Contents                |
| Espressif FW      | nonos-sdk 2.2.1 + 100 (190703)    |
| SSL Support       | All SSL ciphers (most compatible) |

点击 `项目` 菜单，点击 `导出已编译的二进制文件`。稍等片刻， `esp_duck`  目录中将生成 `esp_duck.ino.generic.bin` 文件。

启动 [ESP8266Flasher.exe](https://github.com/nodemcu/nodemcu-flasher/blob/master/Win32/Release/ESP8266Flasher.exe)，点击第二个选项卡 `Config` 中第一行右侧的齿轮（注意不要勾选除第一行之外其他行前的复选框），在打开的选择文件对话框选择上述 `esp_duck.ino.generic.bin` 文件；点击第三个选项卡 `Advanced`，按照如下配置：

| 参数        | 选项   |
| ----------- | ------ |
| Baudrate    | 150200 |
| Flash size  | 4MByte |
| Flash speed | 40MHz  |
| SPI Mode    | DIO    |

然后我们需要对 CJMCU-3212 进行一点处理：先拔出开发板，连通 USB 公头朝左时 ATmega32u 芯片上方的与其他众多触点相独立的两个触点（如果你不知道我指的是哪两个，可以查看 puckk 文档中 [他拍的照片](https://github.com/puckk/CJMCU-3212/blob/master/README.md#2-upload-the-sketch-on-the-esp)），再插入电脑。在保持连通的状态下，在 ESP8266Flasher.exe 的第一个选项卡中的 `COM Port` 选择此时 `设备管理器` 中显示的端口，然后点击右侧 `Flash` 按钮。

正常情况下仅需数秒软件就会获取到 CJMCU-3212 的信息并开始烧录，如果长时间没有反应，可能是两个触点连接出现问题，请调整后重试。

等待烧录完毕后，可以拔出 CJMCU-3212 并断开两触点。

## 5. 烧录 ATmega32u4 程序

使用 Arduino IDE 打开 `atmega_duck/atmega_duck.ino` 文件。

点击 Arduino IDE 顶部 `工具` 菜单，选择第三个集合中第一项 `开发板: "*"`，选择第二个集合中的 `WiFi Duck AVR`，再选择 `CJMCU Beetle` 。

再次点击 `工具` 菜单将出现很多可选参数，请依照下表设置：

| 参数                     | 选项             |
| ------------------------ | ---------------- |
| USB Device ID            | Arduino Leonardo |
| Protocol                 | Serial           |
| Debug                    | Disabled         |
| LED                      | Disabled         |
| LED Pin                  | RX (Serial)      |
| Serial Bridge            | Serial 1         |
| Serial Bridge Enable Pin | SDA (I2C)        |
| Serial Bridge Reset Pin  | SDA (I2C)        |
| Serial Bridge GPIO-0 Pin | SDA (I2C)        |

点击 `项目` 菜单，点击 `上传`。稍等片刻，待显示 `上传成功` 后，WiFiDuck 已经成功的烧录到了 CJMCU-3212 中。

## 6. 完成

拔出后重新插入供电，搜索 WiFi 可以找到 SSID 为 `wifiduck` 的热点，使用密码 `wifiduck` 登录，访问 `192.168.4.1` 即可 [正常使用](https://github.com/SpacehuhnTech/WiFiDuck#usage)。

与 puckk 一样，我希望这个方法能够帮到你 : )

## * 感谢

* [puckk/CJMCU-3212](https://github.com/puckk/CJMCU-3212)
* [spacehuhn/WiFiDuck](https://github.com/SpacehuhnTech/WiFiDuck)
* [nodemcu/nodemcu-flasher](https://github.com/nodemcu/nodemcu-flasher)
* [bhiggins-2@Issue: TRACE +0.109 Timed out waiting for packet header #1](https://github.com/puckk/CJMCU-3212/issues/1#issuecomment-411513077)
* 和看到这里的你！

