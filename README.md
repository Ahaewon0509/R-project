# 602277111 안혜원

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
	 - for(i in 1:nrow(loc)){           # 외부반복: 25개 자치구
	 for(j in 1:length(datelist)){  # 내부반복: 12개월
	 cnt <- cnt + 1               # 반복누적 카운팅
    #---# 요청 목록 채우기 (25 X 12= 300)
    url_list[cnt] <- paste0("http://openapi.molit.go.kr:8081/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTrade?", #한컴자료 call back URL
                            "LAWD_CD=", loc[i,1],         # 지역코드
                            "&DEAL_YMD=", datelist[j],    # 수집월
                            "&numOfRows=", 100,           # 한번에 가져올 최대 자료 수
                            "&serviceKey=", service_key)  # 인증키
  } 
  Sys.sleep(0.1)   # 0.1초간 멈춤
  msg <- paste0("[", i,"/",nrow(loc), "]  ", loc[i,3], " 의 크롤링 목록이 생성됨 => 총 [", cnt,"] 건") # 알림 메시지
  cat(msg, "\n\n")




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
