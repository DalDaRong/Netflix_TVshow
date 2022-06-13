# 넷플릭스 TV쇼 경향 분석
아시아경제에서 활동하며 진행한 첫 번째 프로젝트입니다. 2020년, 2021년 넷플릭스에서 서비스하는 TV쇼 프로그램의 순위와 컨텐츠 정보를 통해 전 세계의 방송 트렌드와 그 흐름을 분석했습니다.
</br>

<img src="https://user-images.githubusercontent.com/96275852/173315504-028cf110-cbf7-4e04-9ceb-80e2ea205f93.png" width="20%" height="20%"></img>
</br>
데이터 출처 : https://flixpatrol.com/
</br>영화, TV쇼의 순위와 컨텐츠 정보를 연도별, 국가별로 공개하는 해외 웹사이트.</br>


## 목적
현재 가장 인기있는 플랫폼 중 하나인 넷플릭스의 방송 트렌드를 파악할 수 있다면, 추후 소비자의 흥미를 끌 프로그램을 예측해 제작, 투자가 가능할 것이라 기대했습니다.

## 기간
2022.12.24 – 2022.12.31 (1주일)

## 팀 구성
본인 외 3인

## 팀 내 역할
데이터 분석과 시각화, DB 연동, PPT작성, 발표

## 사용 기술
python(BeautifulSoup), DBMS

## 개발 과정 

### 1. 스크래핑
데이터를 수집하기 위해 BeautifulSoup을 이용해 스크래핑했습니다. 순위를 보여주는 Score Table과 작품의 정보가 담긴 Detail Table 두 종류의 데이터를 수집했습니다.

#### 1) Score Table
<img src="https://user-images.githubusercontent.com/96275852/173320468-7afa8aa6-1de3-4168-a09c-cd131cae4f43.png" width="80%" height="80%"></img>
작품 순위를 보여줍니다. 서비스된 국가 수와 기간 등을 고려해 점수가 집계되었습니다. 2020년과 2021년도 각각 상위 150개의 데이터를 스크래핑하고, 이후 detail table 작업을 위해 작품의 제목을 리스트에 담았습니다.
```
res = requests.get('https://flixpatrol.com/top10/netflix/world/2021/full/#netflix-2')
soup = BeautifulSoup(res.content, 'xml')
 
netfl2 = soup.find('div',{'id':'netflix-2'})
table1 = netfl2.find('tbody',{'class':'tabular-nums'})
html_table_21_sub = parser.make2d(table1)
#html_table1
```
#### 2) Detail Table
컨텐츠의 제목, 분류, 국가, 제작사, 장르, 키워드의 정보가 담겨있습니다. Score Table에 있는 작품만 스크래핑했습니다.</br>
<img src="https://user-images.githubusercontent.com/96275852/173318309-67e68382-5d0f-4bba-bea7-630f2141fda9.png" width="80%" height="80%"></img>
```
list_all = [] 

for i in range(len(url_re)):
    html_2 = urllib.request.urlopen(url_re[i])
    soup_2 = BeautifulSoup(html_2,"html.parser")
    info_title = soup_2.find("h1",{"class":"mb-3"})
    info_d= soup_2.find("div",{"class":"flex flex-wrap text-sm leading-6 text-gray-500"})
    spans = info_d.find_all("span")

    ready1=[]

    
    #####리스트에 제목넣음#####
    ready1.append(info_title.get_text())
    
    ##### 1초 ? 쉬어가기 zzzZ #####
    time.sleep(1)    

    #####리스트 1행 씩 생성#####
    for t in range(0,len(spans)):
        excp = spans[t].get_text()
        if excp != "|" and excp != "\n\n" and ("/" in excp)==False:
            ready1.append(spans[t].get_text())
```
### 2. 전처리
스크래핑이 완료된 데이터 전처리를 진행했습니다. 사이트의 실수로 Score Table과 Detail Table에 기재된 컨텐츠의 제목이 다른 경우, 스크래핑 과정 중 공백문자가 문자열로 인식된 경우, 데이터 프레임의 정렬이 맞지 않아 카테고리와 상관없는 값들이 입력된 경우 등 다수의 문제를 해결했습니다. 
``` 
    #####문자열 replace, \n 제거 #####
    ready_2 = []
    for j in ready1:
        temp = j.replace("\n","")
        ready_2.append(temp)

        
    ##### 키워드,제작사 없는곳 빈부분 none 채워넣기 #####
    if (len(ready_2)==5 and ready_2[3] == ' Netflix'):
        ready_2.append(None)
        
    #####밀린 (제작사)빈칸 넣어보기#####
    #총길이len 6
    #index 3 ->제작사 
    if (len(ready_2)<=5):
        ready_2.insert(3,None)

    ##### 키워드 빈부분 none 채워넣기 #####
    if (len(ready_2)==5):
        ready_2.append(None)        
        
    ##### 제작사 앞의 빈칸제거 #####       
    if ready_2[3] != None:
        ready_2[3] = ready_2[3].strip()

        
    ##### 인덱스0에 기본키 넣기 #####   
    ready_2.insert(0,i+1)
    
    #####생성된 행 최종 리스트에 넣음 #####            
    list_all.append(ready_2)
```

### 3. DB연동
MariaDB 내에서 데이터 분석, 시각화 작업을 진행했습니다. 파이썬과 동일하게 데이터 테이블을 작성하고, 한 행씩 입력해주었습니다. 생성한 데이터 프레임은 "score_table_2020.csv", "score_table_2021.csv", "detail_table.csv" 파일에 개별 저장했습니다.
* score_table 
```
##### "score_table"에 2020,2021년도 스코어 자료 넣기 #####

Maria_db = pymysql.connect(
    user='root', 
    passwd='1234', 
    host='127.0.0.1', 
    db='project', 
    charset='utf8'
)

cursor = Maria_db.cursor(pymysql.cursors.DictCursor)

# 컬럼이름지정 :  df의 컬럼이름도 데이터베이스와 같아야 함
cols = "`,`".join([str(i) for i in df_main_21.columns.tolist()]) 

# 행 한줄씩 생성
for i,row in df_main_20.iterrows():
    sql = "INSERT INTO `score_table` (`" +cols + "`) VALUES (" + "%s,"*(len(row)-1) + "%s)"
    cursor.execute(sql, tuple(row))
    
# 행 한줄씩 생성
for i,row in df_main_21.iterrows():
    sql = "INSERT INTO `score_table` (`" +cols + "`) VALUES (" + "%s,"*(len(row)-1) + "%s)"
    cursor.execute(sql, tuple(row))    
    
Maria_db.commit()
Maria_db.close()

```

### 4. 시각화를 통한 분석
분석한 내용을 파이그래프, 히스토그램, 워드클라우드로 시각화했습니다.

#### 1) 제작사
<img src="https://user-images.githubusercontent.com/96275852/173328274-353fd82e-104c-4ddb-b3a7-c9cd7142658d.png" width="50%" height="50%"></img>
</br>150개의 TV쇼 중 넷플릭스 자체제작 프로그램이 차지하는 비율은 전체의 72%로 2020년, 2021년 모두 비슷합니다.</br>
<img src="https://user-images.githubusercontent.com/96275852/173327854-3ef73472-e86a-45e6-97fc-91c4ba5cd81f.png" width="50%" height="50%"></img>
</br>대부분의 NETFLIX작품은 주로 75-150위에 포진되어있을뿐 상위에 랭크되지는 못했습니다. 2021년에는 더 많은 NETFLIX 작품이 하위권에 랭크되었습니다.

#### 2) 장르와 키워드
<img src="https://user-images.githubusercontent.com/96275852/173336008-7c6635b7-296e-491b-9fcd-dc52adf4042f.png" width="50%" height="50%"></img>
</br>작품 대다수가 Drama(dramedy)와 Relationship 키워드를 포함하고 있음을 볼 수 있습니다. 1년간 큰 변화를 찾아볼 수 없었습니다. 작품마다 최저1개, 최대 8개의 키워드를 가져 그 수가 불균형했고, 키워드의 선정 기준 또한 명확하지 않다는 점을 고려해야 합니다.

#### 결론
1) NETFLIX 자체제작 TVshow의 퀄리티가 높아 시청자들에게 인기가 있었던 것이 아니라, 단지 서비스하고있는 작품의 수가 많았기 때문이었습니다.
2) 사이트에서 제공하는 데이터의 변별력 문제로 인해, 장르와 키워드만으로는 방송 트렌드를 파악하기 어렵다고 판단했습니다.

## 회고
파이썬 노트북 내에서 모든 작업을 수행할 수 있었지만 배운 내용을 소화하고자 DB를 이용해 진행했습니다. 그러나 DB작업환경을 구축하는데 1-2일이라는 긴 시간이 소요되었고, 이로 인해 일정에 차질이 생겼습니다. 다행히 팀원들과 노력한 끝에, 발표 당일 프로젝트를 완성할 수 있었습니다. 시간이 부족했던 만큼 계획을 보다 체계적으로 세웠다면 보다 여유롭게 진행할 수 있지 않았을까 아쉬움이 있습니다. 특히 시각화 자료를 만들 때 어떤 자료가 필요한지 명확하지 않아 다양하게 작성한 후 필요한 부분을 취사선택했는데, 목적과 용도를 미리 정해놓고 진행했다면 시간을 절약할 수 있었을 것입니다.


