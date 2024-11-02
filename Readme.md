# fir_rtl_eval

### 简介

使用基于python编写的脚本，以[scipy.signal.lfilter](https://www.osgeo.cn/scipy/reference/generated/scipy.signal.lfilter.html)方法作为FIR软件参考计算结果，使用**VCD文件评估**相同输入下**RTL电路计算结果相较于软件结果的MSE**误差

其中：

- 16阶fir电路参考自菜鸟教程文章<Verilog 教程 7.2 Verilog 并行FIR滤波器设计>：[https://www.runoob.com/w3cnote/verilog-fir.html](https://www.runoob.com/w3cnote/verilog-fir.html)

本工程运行需要安装iverilog工具与gtkwave波形查看工具（不看波形可不装）

**以下所有命令基于fir_rtl_eval根目录运行**

---

### RTL代码编译：

#### RTL文件：

- fir文件夹为16阶fir电路代码

#### RTL编译查看波形：

**fir代码**编译：

```powershell
#编译代码，生成执行文件
verilog -o fir_sim ./fir/fir_tb.sv ./fir/fir_guide.v  ./fir/mult_cell.v ./fir/mult_man.v
#生成波形
vvp ./fir_sim
#查看波形
gtkwave ./fir_tb.vcd
```

#### fir文件夹中：

- **cos_data.m**：生成双cos波形叠加后的输出波形作为testbench输入
- **cosx0p25m7p5m12bit.txt**：**cos_data.m**输出的输入数值，**数据为16进制**，可根据testbech需要修改存储格式
- **other files**：RTL代码

**cosx0p25m7p5m12bit.txt**使用方法详见**fir_tb.sv**：

![image-20241102204023944](D:\LLM-IC\fir_rtl_eval\image\image-20241102204023944.png)

---

### Python脚本玩法

评估脚本为**fir_eval.py**

Python需要安装**scipy**包运行lfilter函数：

```powershell
#安装
pip install scipy
#若有uv工具
uv pip install scipy
```

#### 运行示例：

使用下列命令运行fir_eval脚本评估16阶fir电路：

```powershell
python fir_eval.py --coe_dir ./fir_coef.txt --inter_scale 2048  --VCD_dir ./fir_tb.vcd
```

本脚本预留参数接口：

- **python**：当前运行环境的python命令，必填
- **coe_dir**：FIR乘加链系数文件路径，**所存数据建议为已完成使用所定义定点放大系数放大的10进制整数**
- **inter_scale**：小数转定点数放大系数，必填
- **VCD_dir**：VCD文件路径，必填
- **sign_inter**：是否使用了定点整形，默认为False
- **in_bitwidths**：单个数据输入位宽，sign_inter为True时必填
- **out_bitwidths**：单个数据输出位宽，sign_inter为True时必填

本案例**16阶fir电路系数**参考保存自**fir_coef.txt**，数据**已完成定点系数（2^11）放大**，参考为：

![image-20241102212842181](D:\LLM-IC\fir_rtl_eval\image\image-20241102212842181.png)

---

### RTL代码规范

为了保证脚本的正确正常运行，要求RTL代码具有一定的规范：

- fir电路**输入端口**命名：**xin**
- fir电路**输出端口**命名：**yout**
- fir电路**clock**命名：**clk**
- fir电路**输入有效位**：**en**

- fir电路**输出有效位**：**valid**

#### 时序规范：

- **en**信号为高时，**输入可立即有效**
- **valid**信号为高时，**输入需要下一个时钟周期才能有效**

**不规范的代码将导致脚本无法使用**

---

### 实验结果：

基于参考的**16阶fir电路**，**MSE**值：***168443.32587910***

可使用计算精度更高的电路进行实验

---

### 注：

- 根据MSE值结果，本仓库所参考电路计算精度并不理想
- 脚本功能仍不完善，具有隐藏bug的可能性，还需要针对具体应用修补完善

