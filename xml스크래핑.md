## XML을 이용한 스크래핑

### RSS(Really Simple Syndication)
* 뉴스나 블로그 등 업데이트가 빈번한 사이트에서 주로 활용
* 구독자들에게 업데이트된 정보를 용이하게 제공하기 위해 xml기반 정보 표현 제공


```python
from xml.etree import ElementTree
```


```python
tree= ElementTree.parse('rss.xml')
```


```python
root = tree.getroot()
```


```python
import pandas as pd
```


```python
데이터프레임_리스트 = []
for item in root.findall('channel/item/description/body/location/data'):
    # find() 메서드로 element 탐색, text 속성으로 값을 추출
    tm_ef = item.find('tmEf').text
    tmn = item.find('tmn').text
    tmx = item.find('tmx').text
    wf = item.find('wf').text
    데이터프레임 = pd.DataFrame({
        '일시':[tm_ef],
        '최저기온':[tmn],
        '최고기온':[tmx],
        '날씨':[wf],
    })
    데이터프레임_리스트.append(데이터프레임)
날씨정보 = pd.concat(데이터프레임_리스트) #여러개의 엑셀 파일 하나로 합치는 함수 
날씨정보
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>일시</th>
      <th>최저기온</th>
      <th>최고기온</th>
      <th>날씨</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-06-25 00:00</td>
      <td>21</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-06-25 12:00</td>
      <td>21</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-06-26 00:00</td>
      <td>21</td>
      <td>29</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-06-26 12:00</td>
      <td>21</td>
      <td>29</td>
      <td>구름많음</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-06-27 00:00</td>
      <td>22</td>
      <td>29</td>
      <td>구름많음</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-06-29 00:00</td>
      <td>23</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-06-29 12:00</td>
      <td>23</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-06-30 00:00</td>
      <td>22</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-07-01 00:00</td>
      <td>22</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2020-07-02 00:00</td>
      <td>22</td>
      <td>27</td>
      <td>흐림</td>
    </tr>
  </tbody>
</table>
<p>533 rows × 4 columns</p>
</div>




```python
type(날씨정보)
```




    pandas.core.frame.DataFrame




```python
날씨정보.to_csv('날씨정보.csv')
```


```python
엑셀 = pd.ExcelWriter('날씨정보.xlsx') #엑셀로 저장 
날씨정보.to_excel(엑셀, '.', index=False )
엑셀.save()
```


```python
엑셀
```




    <pandas.io.excel._xlsxwriter.XlsxWriter at 0x1c8ba7c5610>




```python
#drop은 세팅한 열을 dataframe내에서 삭제 여부 설정
#inplace는 원본 객체 변경 여부 설정 

날씨정보.reset_index(drop=True, inplace=True)
```


```python
날씨정보.to_json('날씨정보.json')
```


```python
import sqlite3 #데이터 베이스 액세스할 수 있는 라이브러리 
from pandas.io import sql 
import os
```


```python
with sqlite3.connect(os.path.join('.','sqliteDB')) as con: # sqlite DB 파일이 존재하지 않는 경우 파일생성
    try:
        날씨정보.to_sql(name = 'WEATHER_INFO', con = con, index = False, if_exists='append') 
        #if_exists : {'fail', 'replace', 'append'} default : fail
    except Exception as e:
        print(str(e))
    
    query = 'SELECT * FROM WEATHER_INFO'
    데이터프레임1 = pd.read_sql(query, con = con)
```


```python
데이터프레임1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>일시</th>
      <th>최저기온</th>
      <th>최고기온</th>
      <th>날씨</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-06-25 00:00</td>
      <td>21</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-06-25 12:00</td>
      <td>21</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-06-26 00:00</td>
      <td>21</td>
      <td>29</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-06-26 12:00</td>
      <td>21</td>
      <td>29</td>
      <td>구름많음</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-06-27 00:00</td>
      <td>22</td>
      <td>29</td>
      <td>구름많음</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1594</th>
      <td>2020-06-29 00:00</td>
      <td>23</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1595</th>
      <td>2020-06-29 12:00</td>
      <td>23</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1596</th>
      <td>2020-06-30 00:00</td>
      <td>22</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1597</th>
      <td>2020-07-01 00:00</td>
      <td>22</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1598</th>
      <td>2020-07-02 00:00</td>
      <td>22</td>
      <td>27</td>
      <td>흐림</td>
    </tr>
  </tbody>
</table>
<p>1599 rows × 4 columns</p>
</div>




```python
엑셀 = pd.ExcelWriter('날씨정보2.xlsx')
데이터프레임1.to_excel(엑셀, '.', index=False )
엑셀.save()
```


```python
df = pd.read_excel('날씨정보2.xlsx')
```


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>일시</th>
      <th>최저기온</th>
      <th>최고기온</th>
      <th>날씨</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-06-25 00:00</td>
      <td>21</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-06-25 12:00</td>
      <td>21</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-06-26 00:00</td>
      <td>21</td>
      <td>29</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-06-26 12:00</td>
      <td>21</td>
      <td>29</td>
      <td>구름많음</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-06-27 00:00</td>
      <td>22</td>
      <td>29</td>
      <td>구름많음</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1594</th>
      <td>2020-06-29 00:00</td>
      <td>23</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1595</th>
      <td>2020-06-29 12:00</td>
      <td>23</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1596</th>
      <td>2020-06-30 00:00</td>
      <td>22</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1597</th>
      <td>2020-07-01 00:00</td>
      <td>22</td>
      <td>26</td>
      <td>흐리고 비</td>
    </tr>
    <tr>
      <th>1598</th>
      <td>2020-07-02 00:00</td>
      <td>22</td>
      <td>27</td>
      <td>흐림</td>
    </tr>
  </tbody>
</table>
<p>1599 rows × 4 columns</p>
</div>


