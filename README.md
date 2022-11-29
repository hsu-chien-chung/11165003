# 11165003 #
# Rare Events Analysis for High-Frequency Equity Data #

## 作者：Dragos Bozdog, Ionut¸ Florescu, Khaldoun Khashanah, and Jim Wang,<br/>學校：Stevens Institute of Technology,<br/>e-mail: ifloresc@stevens.edu ##

## Abstract ##

### In this work we present a methodology to detect rare events which are defined as large price movements relative to the volume traded. We analyze the behavior of equity after the detection of these rare events. We provide methods to calibrate trading rules based on the detection of these events and illustrate for a particular trading rule. We apply the methodology to tick data for thousands of equities over a period of five days. In order to draw comprehensive conclusions, we group the equities into classes and calculate probabilities of price recovery after these rare events for each class. The methodology that we have developed is based on non-parametric statistics and makes no assumption about the distribution of the random variables in the study. ###

## 將兩函數程式化：<br/>![image](https://user-images.githubusercontent.com/118785456/204456081-c6fa7456-35e8-4941-8128-7b798b084f77.png)<br/>![image](https://user-images.githubusercontent.com/118785456/204456185-7e215b9c-8f8f-40c6-82ff-8edd4a07e9f8.png) ##

# 高頻股票的罕見事件分析－以APPLE股票為例 #

## 載入yahoo! finance 套件
```python
!pip install yfinance
```
## 透過套件來獲取APPLE股票資料並顯示前五筆
```python
import yfinance as yf
apple = yf.Ticker('AAPL')

apple.actions

hist=apple.history(period="1y")
hist=hist.reset_index()
hist.head()
```
![image](https://user-images.githubusercontent.com/118785456/204454504-5be21f37-572e-42f0-8fc3-bfff6d6a1490.png)
## 顯示需要的資料變數
```python
hist[["Close","Volume","Date"]]
```
![image](https://user-images.githubusercontent.com/118785456/204454951-a878e25a-8910-4b14-a776-2c312a43a697.png)
## 交易量的趨勢圖
```python
import matplotlib.pyplot as plt
plt.plot(hist[["Volume"]].iloc[:, 0])
```
![image](https://user-images.githubusercontent.com/118785456/204455451-86bbadc4-4a5b-4e16-b96e-0afc016efbc0.png)
## 計算 ##
![image](https://user-images.githubusercontent.com/118785456/204457353-6b96adce-543c-4aa6-b8d4-5ad14b898ca0.png)<br/>![image](https://user-images.githubusercontent.com/118785456/204458298-ba3895b0-f5e6-4ecb-a1d9-f5d81ecac0e5.png)<br/>![image](https://user-images.githubusercontent.com/118785456/204462017-55df0386-2486-434c-aee9-314bb903645b.png)
```python
import pandas as pd
import numpy as np

k = []
df = []
k_max = []

V0=hist["Volume"].mean()*(len(hist)/4) #用一季的平均交易量來當預設
print(V0) #V0的預設值
data = hist[["Close","Volume","Date"]]
v=0
a=0
b=0
c=0

for i in range(0, data.shape[0]):
  v=0
  for j in range(i,0-1,-1):
    if(v < V0):
      a=a+1
      v=v+data.iloc[j, 1]
      df.append([data.iloc[i, 0]-data.iloc[j, 0],i,j,v])
      k.append(data.iloc[i, 0]-data.iloc[j, 0])
    else:
      break;
  #計算 delta_Pn
  df1=np.asarray(df)
  if (df1.shape[0]==0):
    c = np.max(df1[b:a])
  else:  
    c = np.max(df1[b:a,0])
  k_max.append(c)
  b=a
  
df = np.asarray(df)
m = np.argmax(k)
print(pd.DataFrame(df)) #每一筆交易的價差與累計交易量
print(max(k))  #最大價差
print(df[m,1:].astype(int)) #最大價差的交易時期與累計交易量
```
## V0的預設值
![image](https://user-images.githubusercontent.com/118785456/204459472-8b68d3a0-aad3-4ec0-b51c-cc48218d50c8.png)
## delta_P的值、交易期間與累計交易量
![image](https://user-images.githubusercontent.com/118785456/204460191-2cede98e-e74b-4277-aa34-9dca6b30e550.png)
## 最大交易價差、交易期間與累計交易量
![image](https://user-images.githubusercontent.com/118785456/204460552-b04836ee-21ac-400a-8aaf-cbd4103aba6c.png)
## ![image](https://user-images.githubusercontent.com/118785456/204456081-c6fa7456-35e8-4941-8128-7b798b084f77.png) ##
```python
k_max = np.asarray(k_max) 
print(pd.DataFrame(k_max))  #delta_Pn = 0～251的所有最大價差
```
![image](https://user-images.githubusercontent.com/118785456/204461203-0831eab0-3e3a-49ee-9fcc-37c7caf4076e.png)
## 最大價差出售、購買日期 ##
```python
a = df[m,1].astype(int)
b = df[m,2].astype(int)
print(data.iloc[a,2]) #最大價差出售日期
print(data.iloc[b,2]) #最大價差購買日期
```
![image](https://user-images.githubusercontent.com/118785456/204463466-686fc721-27e2-401b-b29b-267a79306ae7.png)
## 計算 ##
![image](https://user-images.githubusercontent.com/118785456/204456185-7e215b9c-8f8f-40c6-82ff-8edd4a07e9f8.png)
```python
def Y(Data):
  df_dp = pd.DataFrame(Data)
  m = float(df_dp.mean())
  se = float(df_dp.std() / len(hist)**0.5)

  return (m-1.96*se)

Y(Data = df[:,3].round(2))  #Qa(x)
```
## 交易量的下臨界值 ##
![image](https://user-images.githubusercontent.com/118785456/204464379-4c25924b-fd57-4597-8953-704299f406b5.png)

