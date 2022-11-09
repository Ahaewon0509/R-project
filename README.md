# 602277111 안혜원

## `[11월 09일]`
### 1. 지도 그리기
	1. 지도 위에 래스터 이미지 올리기
	- rgdal 패키지를 설치하고 불러오기 -> leaflet()으로 지도 라이브러리 사용을 선언하고,
	addProviderTiles() 옵션으로 지도의 기본 테마를 정함
	- 또한 addPolygon()은 서울시 외곽선을 불러오고, 외곽선의 폭과 색상을 조정할 수 있다.
	- 코드: library(rgdal)    # install.packages("rgdal")
	library(leaflet)  # install.packages("leaflet")
	leaflet() %>% 
	#---# 베이스맵 불러오기
	addProviderTiles(providers$CartoDB.Positron) %>% 
	#---# 서울시 경계선 불러오기
	addPolygons(data = bnd, weight = 3, color= "red", fill = NA) %>% 
	#---# 레스터 이미지 불러오기addRasterImage(raster_high, 
                 colors = colorNumeric(c("blue", "green","yellow","red"), 
                 values(raster_high), na.color = "transparent"), opacity = 0.4)
	
### 2. 요즘 뜨는 지역은 어디일까?
	1. 데이터 준비하기
	- 두 시점 사이의 가격 변화를 알려면 가장 먼저 시간 범위를 설정해야 합니다.
	- load()로 아파트 실거래 자료를 불러오고, st_read()로 서울시 그리드 파일을 읽어온다.
	- st_join()으로 실거래와 그리드를 공간 결합한 후 head()로 결과를 확인하면 됩니다.
	- 코드: #---# [1단계: 데이터 준비]
	setwd(dirname(rstudioapi::getSourceEditorContext()$path)) # 작업폴더 설정
	load("./06_geodataframe/06_apt_price.rdata")     # 실거래 불러오기
	grid <- st_read("./01_code/sigun_grid/seoul.shp")  # 서울시 1km 그리드 불러오기
	apt_price <-st_join(apt_price, grid, join = st_intersects)  # 실거래 + 그리드 공간결합
	head(apt_price, 2)
    

## `[11월 02일]`
### 1. 지역별 평균 가격 구하기
	1. 실거래 + 그리드 데이터 결합
	- load() 함수로 실거래 자료(포인트 데이터)를 불러오고, st_read()로 서울시 그리드 파일을 읽어온 다음 
	st_join()으로 특성이 서로 다른 두 데이터를 결합
	- 압선 결합 결과로 실거래 자료에는 좌표에 해당하는 그리드 ID(apt_price$ID)가 추가되며, 
	aggregate() 함수로 그리드 내에 거래된 평당 거래가(apt_peice$py)를 취합하여 
	평균가격(ked_high)을 구한다.
	
### 2. 평균가격 정보 표시하기
	1. 그리드 + 평균 가격 결합
	- 코드: kde_high <- merge(grid, kde_high,  by="ID")   # ID 기준으로 결합
	library(ggplot2) # install.packages("ggplot2")
	library(dplyr)   # install.packages("dplyr")
	kde_high %>% ggplot(aes(fill = avg_price)) + # 그래프 시각화
	geom_sf() + 
	scale_fill_gradient(low = "white", high = "red")
	
### 3. 지도 경계 그리기
	1. sp형으로 변환과 그리드별 중심 좌표 추출
	- 코드: library(sp) # install.packages("sp")
	kde_high_sp <- as(st_geometry(kde_high), "Spatial")    # sf형 => sp형 변환
	x <- coordinates(kde_high_sp)[,1]  # 그리드 x, y 좌표 추출
	y <- coordinates(kde_high_sp)[,2]
	
	2. 기준 경계 설정
	- l1 <- bbox(kde_high_sp)[1,1] - (bbox(kde_high_sp)[1,1]*0.0001) 
	# 그리드 기준 경계지점 설정
	l2 <- bbox(kde_high_sp)[1,2] + (bbox(kde_high_sp)[1,2]*0.0001)
	l3 <- bbox(kde_high_sp)[2,1] - (bbox(kde_high_sp)[2,1]*0.0001)
	l4 <- bbox(kde_high_sp)[2,2] + (bbox(kde_high_sp)[1,1]*0.0001)


## `[10월 26일]`
### 1. 주소와 좌표 결합하기
	1. 데이터 불러오기
	- 06_geodataframe 새로운 폴더 코드 작성 -> geo_dataframe.R 새로운 파일 생성
	- 코드: setwd(dirname(rstudioapi::getSourceEditorContext()$path))
	load("./04_preprocess/04_preprocess.rdata")    # 주소 불러오기
	load("./05_geocoding/05_juso_geocoding.rdata") # 좌표 불러오기

### 2. 주소와 좌표 결합하기
	1. 주소 + 좌표 결합
	- 서로 다른 데이터프레임 안에 공통된 항목이 있으면 하나로 결합할 수 있음
	- 데이터를 결합할 때는 left_join() 함수를 사용
	
### 3. 지오 데이터프레임 만들기
	1. 지오 데이터프레임 생성하기
	- 코드: library(sp)    # install.packages('sp')
	coordinates(apt_price) <- ~coord_x + coord_y    # 좌표값 할당
	proj4string(apt_price) <- "+proj=longlat +datum=WGS84 +no_defs" # 좌표계(CRS) 정의
	library(sf)    # install.packages('sf')
	apt_price <- st_as_sf(apt_price)     # sp형 => sf형 변환
	
	2. 데이터프레임 시각화
	- 코드: plot(apt_price$geometry, axes = T, pch = 1)        # 플롯 그리기 
	library(leaflet)   # install.packages('leaflet')   # 지도 그리기
	leaflet() %>% 
	addTiles() %>% 
	addCircleMarkers(data=apt_price[1:1000,], label=~apt_nm) # 일부분(1000개)만 그리기

## `[10월 12일]`
### 1. 지오 코딩 준비하기
	1.카카오 로컬 API 키 발급받기
	- developers.kakao.com 사이트 접속 -> 카카오계정으로 로그인 -> [내 애플리케이션] 메뉴 클릭
	-> [애플리케이션 추가하기]: 앱 이름, 이름+사업자명 입력 후 저장 - > "카카오맵이 제공하
	REST API 키" 발급확인
	
### 2. 새로운 파일, 폴더 생성
        1. 05_geocoding 새로운 폴더 코드 작성
	- 코드: juso_geocoding <- rbindlist(add_list)   # 리스트를 데이터프레임 변환
	juso_geocoding$coord_x <- as.numeric(juso_geocoding$coord_x) # 좌표값 숫자형 변환
	juso_geocoding$coord_y <- as.numeric(juso_geocoding$coord_y)
	juso_geocoding <- na.omit(juso_geocoding)   # 결측치 제거
	dir.create("./05_geocoding")   # 새로운 폴더 생성
	save(juso_geocoding, file="./05_geocoding/05_juso_geocoding.rdata") # 저장
	write.csv(juso_geocoding, "./05_geocoding/05_juso_geocoding.csv")

	2. 지오 코딩 준비하기: kakao_key에는 인증받은 REST API 키 입력
	- 코드: add_list <- list()   # 빈 리스트 생성
	cnt <- 0             # 반복문 카운팅 초기값 설정
	kakao_key = "5ca06cab87edc59da94313938eb4975f"       # 인증키

### 3. 좌표계와 지오 데이터 포맷
        1. 좌표계(CRS)란 무엇일까?
	- 지구는 불규칙한 타원체라서 실제 좌푯값을 표현하려면 '투영과정'을 거쳐 보정해야 하며, 이때 보정의 기준이 "좌표계"라고 함

## `[10월 05일]`
### 1. 통합 데이터 저장하기
	1. 03_integrated 새로운 폴더 생성
	- 03_integrated -> 새로운 폴더를 생성하고, save(), write.csv()로 데이터를 RDATA와 CSV 형식으로 저장
	
### 2. 불필요한 정보 지우기
        1. 수집한 데이터 불러오기
	- 코드: pre_process.R 파일 생성
	setwd(dirname(rstudioapi::getSourceEditorContext()$path))
	options(warn=-1)
	 
	load("./03_integrated/03_apt_price.rdata") #실거래 자료 불러오기
	head(apt_price, 2) #자료확인
	
	2. 결측값과 공백 제거하기
	- NA제거 코드
	-> NA로 된 데이터는 계산할 수 없으므로 제거, 대체해야 함
	-> 먼저 NA값이 있는지 확인 -> is.na()함수를 사용하는데, table() 함수를 함께 사용하면 NA가 몇개가 포함되어 있는지 알 수 있음
	
	코드: table(in.na(apt_price)) #결측값 확인
	
	apt_price <- na.omit(apt_price) #결측값 제거
	table(is.na(apt_price)) #결측값 확인




## `[9월 28일]`
### 1. 자료 요청하고 응답받기
	1. 요청 URL
	- url_list를 xmlTreeParse()로 보내고, 응답 결과인 XML을 raw_data[[i]]에 저장
	- xmlRoot()로 XML의 루트 노드만 추출하여 임시 저장소인 root_Node[[i]]에 저장
	
### 2. 전체 거래 건수 확인
	1. 거래 건수 확인
	items <- root_Node[[i]][[2]][['items']]
	size <- xmlSize(items)
	- 전체 거래 내역은 root_Node[[i]][[2]][['items']] 코드로 추출함
	- xmlSize() 함수로 전체 거래 건수가 몇 개인지도 알아냄





## `[9월 21일]`
### 1. Rstudio 작업 디렉토리 설정
	1. 방법
	 - Tools -> Global Options -> 
	 General에 있는 Default working directory(when not in a project에 
	 "C:/R_project" 경로 변경하면 됨
	 
### 2. 요청목록 생성
	1. 요청목록 만들기
	 - url_list <- list()
	 cnt <-0
	2. 요청목록 채우기
	 - 코드: for(i in 1:nrow(loc)){           # 외부반복: 25개 자치구
	 for(j in 1:length(datelist)){  # 내부반복: 12개월
	 cnt <- cnt + 1               # 반복누적 카운팅
    #---# 요청 목록 채우기 (25 X 12= 300)
    url_list[cnt] <- paste0("http://openapi.molit.go.kr:8081/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTrade?", #한컴자료 call back URL
                            "LAWD_CD=", loc[i,1],         # 지역코드
                            "&DEAL_YMD=", datelist[j],    # 수집월
                            "&numOfRows=", 100,           # 한번에 가져올 최대 자료 수
                            "&serviceKey=", service_key)  # 인증키
  } Sys.sleep(0.1)   # 0.1초간 멈춤
  msg <- paste0("[", i,"/",nrow(loc), "]  ", loc[i,3], " 의 크롤링 목록이 생성됨 => 총 [", cnt,"] 건") # 알림 메시지             cat(msg, "\n\n")
} 



## `[9월 14일]`
### 1. API 크롤러 만들기
	1. 작업 폴더 설정
	 - C 드라이브 "R-project" 폴더 생성
	 - 이지퍼블리싱 홈페이지 접속 -> 소스 파일 내려받기(코드 확인시 사용)
	 - 작업 폴더에 "data_coll.R" -> 새로운 R 스트립트 파일 만든 후 코드 작성
	 
* data_coll.R 코드 작성
```
install.packages("rstudioapi") #rstudioapi 설치
setwd((dirname(rstudioapi::getSourceEditorContext()$path)) # 작업 폴더 설정
getwd() # 작업 폴더 확인
```
### 2. 비주얼스튜디오코드와 연동
       1. 방법: visualstudiocode -> R 설치(install) -> setting -> rpath 검색 -> C 드라이브 "C:\program Files\R\R-4.2.1\bin\x64"
      
## `[9월 7일]`
### 1. API 인증키 얻기
	1. 공공데이터포털 접속(www.data.go.kr)
	
	2. 방법:  
	 - 1단계 : api 활용신청하기 -> "국토교통부_아파트매매 실거래자료" -> 활용신청 클릭
	 - 2단계 : 승인 및 인증키 확인하기 -> 인증키 발급 확인 -> "아파트 매매 신고정보 조회 기술문서.hwp" 다운로드 내용 확인
	
	3. 요청 URL 만들기: 요청 URL에 serviceKey=###인증키###(발급받은 인증키 입력)
