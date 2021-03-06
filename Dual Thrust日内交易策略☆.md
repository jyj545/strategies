
> 策略名称

Dual Thrust日内交易策略☆

> 策略作者

Hukybo

> 策略描述

# 前言
Dual Thrust直译为“双重推力”，是上个世纪80年代由Michael Chalek开发的一个交易策略，曾经在期货市场风靡一时。由于策略本身思路简单，参数很少，因此可以适应于很多金融市场，正是因为简单易用和普适性高的特点，得到了广大交易者的认可流传至今。
[点击阅读更多内容](https://www.fmz.com/bbs-topic/4757)

> 策略参数



|参数|默认值|描述|
|----|----|----|
|Cycle|5|周期|
|Ks|true|Ks|
|Kx|2|Kx|
|FuturesCode|rb000|期货代码|


> 源码 (python)

``` python
# 回测配置 
'''backtest
start: 2019-01-01 00:00:00
end: 2021-01-01 00:00:00
period: 1h
basePeriod: 1h
balance: 10000
slipPoint: 2
exchanges: [{"eid":"Futures_CTP","currency":"FUTURES"}]
'''


# 定义全局变量
mp = 0  													# 用于控制虚拟持仓
last_bar_time = 0  										# 用于判断K线时间
up_line = 0  											# 上轨
down_line = 0  											# 下轨

# 策略参数
Ks = 3
Kx = 2
Cycle = 5

# 策略主函数
def onTick():
    global mp, last_bar_time, up_line, down_line 		# 引入全局变量
    exchange.SetContractType('rb000')  					# 订阅期货品种
    bar_arr = exchange.GetRecords()  					# 获取K线列表
    # 如果没有获取到K线数据或者K线数据太短就返回
    if not bar_arr or len(bar_arr) < 5:
        return  
    last_bar = bar_arr[len(bar_arr) - 1]  				# 最新的K线
    last_bar_close = last_bar['Close']  				# 最新K线的收盘价
    if last_bar_time != last_bar['Time']:  				# 如果产生了新的K线
        hh = TA.Highest(bar_arr, Cycle, 'High')  			# 最高价
        hc = TA.Highest(bar_arr, Cycle, 'Close')  			# 最高的收盘价
        ll = TA.Lowest(bar_arr, Cycle, 'Low')  				# 最低价
        lc = TA.Lowest(bar_arr, Cycle, 'Close')  			# 最低的收盘价
        Range = max(hh - lc, hc - ll)  					# 计算范围
        up_line = _N(last_bar['Open'] + 3 * Range)  	# 计算上轨
        down_line = _N(last_bar['Open'] - 2 * Range)	# 计算下轨
        last_bar_time = last_bar['Time']  				# 更新最后时间戳
    if mp == 0 and last_bar_close >= up_line:
        exchange.SetDirection("buy")  					# 设置交易方向和类型
        exchange.Buy(last_bar_close, 1)  				# 开多单
        mp = 1  											# 设置虚拟持仓有多单
    if mp == 0 and last_bar_close <= down_line:
        exchange.SetDirection("sell")  					# 设置交易方向和类型
        exchange.Sell(last_bar_close - 1, 1)  			# 开空单
        mp = -1  											# 设置虚拟持仓有空单
    if mp == 1 and last_bar_close <= down_line:
        exchange.SetDirection("closebuy")  				# 设置交易方向和类型
        exchange.Sell(last_bar_close - 1, 1)  			# 平多单
        mp = 0  											# 设置虚拟持仓空仓
    if mp == -1 and last_bar_close >= up_line:
        exchange.SetDirection("closesell")  				# 设置交易方向和类型
        exchange.Buy(last_bar_close, 1)  				# 平空单
        mp = 0  											# 设置虚拟持仓空仓

# 程序入口        
def main():
    while True:  										# 进入循环模式
        onTick()  										# 执行策略主函数
        Sleep(1000)  										# 休眠1秒


```

> 策略出处

https://www.fmz.com/strategy/177504

> 更新时间

2021-01-09 16:36:09
