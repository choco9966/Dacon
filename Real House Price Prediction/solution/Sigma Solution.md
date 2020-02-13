## Sigma Solution 

출처 :  https://dacon.io/competitions/official/21265/codeshare/443 

**전처리와 외부데이터를 활용하는게 배울 점**

- 공공데이터 : 한강 거리 , 공원이름 및 거리, 학교의 학군 및 거리, 초등학교5학년의 학생수 

- 아파트 이름을 활용 

  ```python
  '''
  사전설치 conda install -c conda-forge scrapy pip install -U selenium 이용 사이트 : 직방(
  https://www.zigbang.com
  ) 최종 파일 : apt_name_company_trte.csv
  '''
  
  ###먼저 실행
  apt_info= train.groupby(["apartment_id","latitude","longitude"], as_index=False).count()
  apt_info =apt_info[["apartment_id"]]
  apt_info["apt_name"] = None # apt_name 열 생성
  def get_name(id):
      from selenium import webdriver
      browser = webdriver.Chrome("chromedriver")
      try : 
          browser.get("https://www.zigbang.com/apt/complex/" + str(id))
          import time
          time.sleep(2)
          html = browser.find_element_by_xpath('//*').get_attribute('outerHTML')
          from scrapy.selector import Selector
          selector = Selector(text=html)
          company = selector.xpath('//*[@id="react-danjimap"]/div/div[2]/div[3]/div/div[2]/div[3]/div[1]/div/div/div[1]/text()')[0].extract()
          browser.quit()
      except:
          browser.quit()
          company = None
      return company
      
  for i in range(len(apt_info)):
      id = apt_info["apartment_id"][i]
      apt_info["apt_name"][i] = get_name(id)
      
  apt_info["apt_name"][apt_info["apartment_id"]==5716]= "우신모라"
  apt_info[apt_info["apt_name"].isnull()]  
  #위 크롤링 결과에서 null 값은 3개였음. 다시 실행시킬때마다 달라질 수 있는 점 주의해주세요
  apt_null =  ['반포삼호가든3', '서초 우성1', '구파발 어울림'] #수동으로 찾음
  a=0
  for i in range(len(apt_info)):
      if pd.isna(apt_info["apt_name"][i])==True:
          apt_info["apt_name"][i] = apt_null[a]
          a = a+1
  from collections import OrderedDict
  # Ready for data
  name = OrderedDict()
  
  a = apt_info.loc[:,"apartment_id"]
  b = apt_info.loc[:, "apt_name"]
  key = list(a)
  value = list(b)
  name = dict(zip(key,value)) 
  
  train["apt_name"] = None
  train["apt_name"] = train["apartment_id"].apply(lambda x :name.get(x))
  test["apt_name"] = None
  test["apt_name"] = test["apartment_id"].apply(lambda x :name.get(x))
  #train 기준으로 apt_name을 찾았기때문에 test에 적용시키면 null인 값이 생길 것임 따라서 null값만 
  #빼내서 다시 크롤링함
  test_tmp = test[test["apt_name"].isnull()].groupby("apartment_id",as_index=False).count()
  test_tmp = test_tmp[["apartment_id","apt_name"]]
  test_tmp["apt_name"] = test_tmp["apartment_id"].apply(lambda x: get_name(x)) #test의 null값 크롤링
  apt_info = pd.concat([apt_info, test_tmp], axis=0)
  apt_info.to_csv("apt_name_company_trte.csv",encoding="utf8",index=False)
  ```

- 결측치 채우기 (네이버 부동산 크롤링)

  - 위에서 이름을 찾았기때문에 가능한 접근방법이었음 

  ```
  #먼저 아파트 이름 필요
  #1. 아파트 아이디로 이름 먼저 찾고 (위에서 찾음)
  #2. 이름을 가지고 네이버 부동산에서 찾음
  from collections import OrderedDict
  # Ready for data
  name = OrderedDict()
  
  a = apt_info.loc[:,"apartment_id"]
  b = apt_info.loc[:, "apt_name"]
  key = list(a)
  value = list(b)
  name = dict(zip(key,value)) 
  
  train["apt_name"] = None
  train["apt_name"] = train["apartment_id"].apply(lambda x :name.get(x))
  test["apt_name"] = None
  test["apt_name"] = test["apartment_id"].apply(lambda x :name.get(x))
  from scrapy.selector import Selector
  from selenium import webdriver
  import time
  def heat_null(name):
      browser = webdriver.Chrome("chromedriver")
      try : 
          browser.get("https://land.naver.com/") #네이버 부동산
          search = browser.find_element_by_css_selector('#queryInputHeader').send_keys(name)
          # 또는 search = browser.find_element_by_xpath('//*[@id="queryInputHeader"]').send_keys(name)
          browser.find_element_by_class_name('search_button').click() #입력 후 버튼 클릭
          browser.find_element_by_class_name('complex_link').click() #단지정보 클릭
          html = browser.find_element_by_xpath('//*').get_attribute('outerHTML')
          time.sleep(5)
          selector = Selector(text=html)
          time.sleep(5)
          heat_type = selector.xpath('//*[@id="detailContents1"]/div[1]/table/tbody/tr[5]/td/text()')[0].extract()
          browser.quit()
      except:
          browser.quit()
          heat_type = None
      return heat_type
  train_tmp = train[train["heat_type"].isnull()]
  train_tmp = train_tmp.groupby("apartment_id",as_index=False).mean()
  train_tmp["apt_name"] = train_tmp["apartment_id"].apply(lambda x : name.get(x))
  test_tmp = test[test["heat_type"].isnull()]
  test_tmp = test_tmp.groupby("apartment_id",as_index=False).mean()
  test_tmp["apt_name"] = test_tmp["apartment_id"].apply(lambda x : name.get(x))
  train_tmp["apt_name"][train_tmp["apt_name"]=="장위뉴타운꿈의숲코오롱하늘채"] = "꿈의숲코오롱하늘채"
  test_tmp["apt_name"][test_tmp["apt_name"]=="장위뉴타운꿈의숲코오롱하늘채"] = "꿈의숲코오롱하늘채"
  #바꾸지 않으면 검색이 안됨
  train_tmp["heat_type"] = None #heat_type 채우기   train 기준으로 함
  for i in range(len(train_tmp)):
      train_tmp["heat_type"][i] = heat_null(train_tmp["apt_name"][i])
  #null 값 확인 했을 때 다 개별난방, 도시가스 였음
  # 이번 크롤링 또한 크롤링마다 null 값이 다를 수 있음
  #개별난방, 도시가스 / 개별난방, 도시가스/개별난방, 도시가스/개별난방, 도시가스
  #/개별난방, 도시가스/ 개별난방, 도시가스/개별난방, 도시가스/개별난방, 도시가스/개별난방, 도시가스
  for i in range(len(train_tmp)):
      if train_tmp["heat_type"][i] == None:
          train_tmp["heat_type"][i] = "개별난방, 도시가스"
  from collections import OrderedDict #train의 heat_type 딕셔너리
  
  # Ready for data
  df = OrderedDict()
  
  a = train_tmp.loc[:,"apartment_id"]
  b = train_tmp.loc[:, "heat_type"]
  key = list(a)
  value = list(b)
  df = dict(zip(key,value)) ##g히트타입
  test_tmp["heat_type"] = test_tmp["apartment_id"].apply(lambda x : df.get(x))
  input_null=["지역난방, 열병합","개별난방, 도시가스","개별난방, 도시가스"] #테스트에서 heat_type을 수동으로찾아서 넣음
  a = 0
  for i in range(len(test_tmp)):
        if test_tmp['heat_type'][i]==None:
          test_tmp['heat_type'][i] = input_null[a]
          a = a+1 
  #train, test의 heat_type을 채웠으면 , (콤마)로 분리하여 따로 열을 구분해주는 것이 필요함
  train_tmp["heat_type1"],train_tmp["heat_fuel1"] = train_tmp["heat_type"].str.split(',',1).str
  test_tmp["heat_type1"],test_tmp["heat_fuel1"] = test_tmp["heat_type"].str.split(',',1).str
  train_tmp["heat_type1"] = train_tmp["heat_type1"].replace("개별난방","individual")
  train_tmp["heat_type1"] = train_tmp["heat_type1"].replace("중앙난방","central")
  train_tmp["heat_type1"] = train_tmp["heat_type1"].replace("지역난방","district")
  train_tmp["heat_fuel1"] = train_tmp["heat_fuel1"].str.replace("도시가스","gas")
  train_tmp["heat_fuel1"] = train_tmp["heat_fuel1"].str.replace("열병합","cogeneration")
  test_tmp["heat_type1"] = test_tmp["heat_type1"].replace("개별난방","individual")
  test_tmp["heat_type1"] = test_tmp["heat_type1"].replace("중앙난방","central")
  test_tmp["heat_type1"] = test_tmp["heat_type1"].replace("지역난방","district")
  test_tmp["heat_fuel1"] = test_tmp["heat_fuel1"].str.replace("도시가스","gas")
  test_tmp["heat_fuel1"] = test_tmp["heat_fuel1"].str.replace("열병합","cogeneration")
  
  train_tmp["heat_fuel1"] = train_tmp["heat_fuel1"].str.strip() #공백 생기는 거 제거
  test_tmp["heat_fuel1"] = test_tmp["heat_fuel1"].str.strip()
  from collections import OrderedDict
  #train
  train_heat = OrderedDict()
  
  a = train_tmp.loc[:,"apartment_id"]
  b = train_tmp.loc[:, "heat_type1"]
  key = list(a)
  value = list(b)
  train_heat = dict(zip(key,value)) ##히트타입
  
  # Ready for data
  train_fuel = OrderedDict()
  
  a = train_tmp.loc[:,"apartment_id"]
  b = train_tmp.loc[:, "heat_fuel1"]
  key = list(a)
  value = list(b)
  train_fuel = dict(zip(key,value)) ##히트fuel
  
  #test
  test_heat = OrderedDict()
  
  a = test_tmp.loc[:,"apartment_id"]
  b = test_tmp.loc[:, "heat_type1"]
  key = list(a)
  value = list(b)
  test_heat = dict(zip(key,value)) ##히트타입
  
  # Ready for data
  test_fuel = OrderedDict()
  
  a = test_tmp.loc[:,"apartment_id"]
  b = test_tmp.loc[:, "heat_fuel1"]
  key = list(a)
  value = list(b)
  test_fuel = dict(zip(key,value)) ##히트fuel
  for i in range(len(train_tmp)):
      id =  train_tmp["apartment_id"][i]
      train["heat_fuel"][train["apartment_id"]==id] = train_tmp["heat_fuel1"][i]
  for i in range(len(test_tmp)):
      id =  test_tmp["apartment_id"][i]
      test["heat_fuel"][test["apartment_id"]==id] = test_tmp["heat_fuel1"][i]
      
  # heat_fuel 
  train_tmp2 = train[train["heat_fuel"].isnull()] #heat_fuel null 인거
  test_tmp2 = test[test["heat_fuel"].isnull()]
  
  train_tmp2 = train_tmp2.groupby("apartment_id",as_index=False).mean()
  train_tmp2["apt_name"] = train_tmp2["apartment_id"].apply(lambda x :name.get(x))
  
  test_tmp2 = test_tmp2.groupby("apartment_id",as_index=False).mean()
  test_tmp2["apt_name"] = test_tmp2["apartment_id"].apply(lambda x :name.get(x))
  train_tmp2["heat_fuel"] = None
  test_tmp2["heat_fuel"] = None
  for i in range(len(train_tmp2)):
      train_tmp2["heat_fuel"][i] = heat_null(train_tmp2["apt_name"][i])
  input_null=['중앙난방,도시가스','개별난방,도시가스','개별난방,도시가스','지역난방,열병합','개별난방,도시가스']
  #없는 것 확인 후 수동으로 입력
  a = 0
  for i in range(len(train_tmp2)):
        if train_tmp2['heat_fuel'][i]==None:
          train_tmp2['heat_fuel'][i] = input_null[a]
          a = a+1 
  train_tmp2["heat_type1"],train_tmp2["heat_fuel1"] = train_tmp2["heat_fuel"].str.split(',',1).str #분리 후
  train_tmp2["heat_fuel1"] = train_tmp2["heat_fuel1"].str.replace("도시가스","gas")
  train_tmp2["heat_fuel1"] = train_tmp2["heat_fuel1"].str.replace("열병합","cogeneration") #이름 바꾸어 줌
  
  for i in range(len(train_tmp2)):
      id =  train_tmp2["apartment_id"][i]
      train["heat_fuel"][train["apartment_id"]==id] = train_tmp2["heat_fuel1"][i] #train에 heat_fuel 공백 채우기
  #test도 채우기
  train_fuel1 = OrderedDict()
  
  a = train_tmp2.loc[:,"apartment_id"]
  b = train_tmp2.loc[:, "heat_fuel1"]
  key = list(a)
  value = list(b)
  train_fuel1 = dict(zip(key,value)) ##g히트타입
  test_tmp2["heat_fuel"] = test_tmp2["apartment_id"].apply(lambda x : train_fuel1.get(x))
  
  for i in range(len(test_tmp2)):
      id =  test_tmp2["apartment_id"][i]
      test["heat_fuel"][test["apartment_id"]==id] = test_tmp2["heat_fuel1"][i]
  # 여기까지 heat들어간 열 다 채운줄 알았는데 heat_fuel에 ' - ' 라는 값을 보지 못하였음..
  # 아래는 크롤링이 아닌 heat_type과 heat_fuel 의 관계를 통해 채워넣음
  for_fuel = train[train["heat_fuel"]!='-'] #-이 아닌 것 
  for_fuel = for_fuel[(for_fuel[['key']] == for_fuel[['heat_type', 'key']].groupby('heat_type').transform(max)).squeeze().tolist()]
  for_fuel = for_fuel.loc[:, ['heat_type', 'heat_fuel']]
  
  from collections import OrderedDict
  
  df_fuel = OrderedDict()
  
  a = for_fuel.loc[:,"heat_type"]
  b = for_fuel.loc[:, "heat_fuel"]
  key = list(a)
  value = list(b)
  df_fuel = dict(zip(key,value)) ## 방 개수 딕셔너리
  
  isin = pd.DataFrame()
  isin1 = pd.DataFrame()
  
  isin['fuel'] = train["heat_type"].apply(lambda x : df_fuel.get(x))
  isin1['fuel'] = test["heat_type"].apply(lambda x : df_fuel.get(x))
  
  train['fuel'] = isin['fuel']
  test['fuel'] = isin1['fuel']
  train['heat_fuel'] = np.where((train['heat_fuel']=='-') == True,
                                             train['fuel'], train['heat_fuel'])
  test['heat_fuel'] = np.where((test['heat_fuel']=='-') == True,
                                             test['fuel'], test['heat_fuel'])
  
  train = train.drop(['fuel'], axis=1)
  test = test.drop(['fuel'], axis=1)
  
  train["heat_fuel"] = train["heat_fuel"].apply(lambda x : x.strip()) #공백 제거
  test["heat_fuel"] = test["heat_fuel"].apply(lambda x : x.strip())
  train_real_heat_all = train[["apartment_id","heat_type","heat_fu
  el"]]
  test_real_heat_all = test[["apartment_id","heat_type","heat_fuel"]]
  
  train_real_heat_all.to_csv("train_real_heat_all.csv", index=False)
  test_real_heat_all.to_csv("test_real_heat_all.csv", index=False)
  ```

- 재건축 파생변수 및 허용 용적률 
  
  - **Sigma's view** :  재건축 여부 기준 용적률 일반적으로 아파트의 기존 용적률와 허용 용적률 차이가 클수록 재건축 때 사업성이 높다. 기존 용적률이 높은 단지는 건물을 더이상 높일 수가 없어 상대적으로 사업성이 떨어진다. 서울지여 아파트의 경우 대략적으로 기존 용적률이 180%를 넘으면 재건축 사업이 쉽지 않다 단지크기 : 단지 내 세대수가 많을 수록 재건축 사업성이 높다. 그 기준은 1000세대로 한다. 준공년도 : 아파트 연식이 오래될수록 재건축 확률이 높다. 그 기준은 최근(2018년) 법정기준으로 35년으로 한다. 파생변수(공공데이터 사용) usage_area_name : 용도지역 이름 재건축여부(rebuilding) 공공데이터 사용 기존용적률 : '직방' 웹크롤링 용도지역(허용용적률) : 허용용적률은 용도지역별로 다르다. 따라서 '서울시 용도지역' 데이터를 가져온다. 이용사이트 : 서울열린데이터광장 (http://data.seoul.go.kr/dataList/datasetView.do?infId=OA-13158&srvType=S&serviceKind=1&currentPageNo=1) 이용 csv : 서울시 주거지역 위치정보 (좌표계_ WGS1984).csv 이용방법 : 서울시 데이터에 '위경도, 도시계획사항 명칭' 열을 사용하여 용도지역 별 허용용적률을 붙인다. 새롭게 생성된 데이터 목록 s_train_rebuilding_part3.csv s_test_rebuilding_part3.csv (출처 :  https://dacon.io/competitions/official/21265/codeshare/443 )