---
layout: post
title:  "Data Processing in Shell"
date:   2021-10-07
categories: 데이터엔지니어
tags: 데이터엔지니어
image: /post_img/ubuntu.jpg
---

#  Shell에서 하는 전처리


### Downloading data using curl

`curl` - Client for URLs

서버와 데이터를 주고 받는데 사용

주로 HTTP 사이트 및 FTP 서버에서 데이터를 다운로드하는데 자주 사용



curl 이 잘 다운로드 됬는지 확인하는 커맨드

```shell
man curl

------------------
curl command not found.  -> 잘다운로드 되지 않았을 때


설치되었을 시, 
Name, Synopsis, description등의 내용이 출력

줄을 넘기고 싶을땐 "Enter" 종료할 땐, "q"
```



기본 curl 명령어

```shel
curl [option flags] [URL ]
```

curl은 HTTP,  HTTPS,  FTP,  SFTP 등 다양하게 지원



```shell
curl -O https://websitename.com/datafilename.txt

-----------------------------------
다른 이름으로 저장하고 싶으면 -o (소문자)
curl -o renamedatafilename.txt https://websitename.com/datafilename.txt
```



와일드 카드 사용하여 한번에 다운로드도 가능

```shell
curl -O https://websitename.com/datafilename*.txt

curl -O https://websitename.com/datafilename[001-100].txt

curl -O https://websitename.com/datafilename[001-100:10].txt
```



-L : 300오류 코드가 발생하면 리디렉션

-C : 완료 전에 종료 된 경우 이전 파일 전송을 재개

```shell
curl -L -O -C https://websitename.com/datafilename[001-100].txt
```



### Downloading data using Wget

`Wget` : World Wide Web and get

curl 에 비해 다목적으로, 단일 파일, 전체 폴더, 웹페이지를 다운로드 가능

여러 파일을 재귀적으로 다운로드 가능



wget이 설치된 위치 반환

```shell
which wget

---------------
설치되지않았다면, 아무것도 반환하지않음.
```



설치방법

```shell
#리눅스
sudo apt-get install wget

#Mac
brew install wget

#윈도우 
사이트를 통해 gnuwin32 패키지 일부 설치
```



```shell
wget [option flags] [URL]
```



-b : 다운로드가 백그라운드에서 실행

-q : 출력을 해제하여 공간 절약

-c : 다른 프로그램에 의해 이전에 깨진 다운로드를 완료



```shell
wget -bqc https://websitename.com/datafilename.txt
```



### Advanced downloading using Wget 



파일에있는 내용 다운로드

```shell
cat url_list.txt

wget -i url_list.txt
```



다운로드 제한폭 설정

`--limit-rate` ( 초당 속도 )

```shell
wget --limit-rate={rate}k {file_location}

wget --limit-rate=200k -i url_list.txt
```



대기시간 설정

```shell
wget --wait={second} {file_location}

wget --wait=2.5 -i url_list.txt
```



### Getting started with csvkit



설치방법

```shell
pip install csvkit
```



설명서 보는방법

```shell
in2csv --help
in2csv -h
```



```shell
in2csv SpotifyData.xlsx > SpotifyData.csv

-----------------------------------------
# 첫번째 시트 출력 and 저장 x
in2csv SpotifyData.xlsx 

> SpotifyData.xlsx 

-----------------------------------------
in2csv -n SpotifyData.xlsx

in2csv SpotifyData.xlsx --sheet "Worksheet1_Popularity" > Spotify_Popularity.csv
```



`csvlook`

```shell
# csvlook docs 출력
csvlook -h
```



```shell
# 출력
csvlook Spotify_Popularity.csv
```



```shell
# 다양하게 묘사
csvstat Spotify_Popularity
```



### Filtering data using csvkit

```shell
csvcut -h

csvcut -n Spotify_MusicAttributes.csv # column header 출력

csvcut -c 1 Spotify_MusicAttributes.csv

csvcut -c "track_id" Spotify_MusicAttributes.csv

csvcut -c 2,3 Spotify_MusicAttributes.csv

csvcut -c "danceability","duration_ms" Spotify_MusicAttributes.csv
```



```shell
csvgrep -h

csvgrep -c "track_id" -m 5RCPsfzmEpTXMCTNk7wEfQ Spotify_MusicAttributes.csv
csvgrep -c 1 -m 5RCPsfzmEpTXMCTNk7wEfQ Spotify_MusicAttributes.csv
```



### Stacking data and chaining commands with csvkit

```shell
csvstack -h

# column이 같을때
csvstack Spotify_Rank6.csv Spotify_Rank7.csv > Spotify_AllRanks.csv

csvstack -g "Rank6","Rank7" Spotify_Rank6.csv Spotify_Rank7.csv > Spotify_AllRanks.csv

csvstack -g "Rank6","Rank7" -n "source" Spotify_Rank6.csv Spotify_Rank7.csv > Spotify_AllRanks.csv

# 둘다 실행되지만, 두번째 명령은 첫번째 명령이 실행된 후에만 실행
csvlook SpoitfyData_All.csv; csvstat SpotifyData_All.csv

csvlook SpotifyData_all.csv && csvstat SpotifyData_All data.csv
csvcut -c "track_id","danceability" Spotify_Popularity.csv | csvlook
```



### Pulling data from databases



`sql2csv` : 다양한 인기 SQL 데이터베이스를 지원



```shell
sql2csv -h

sql2csv --db "sqlite:///SpotifyDatabase.db" \
		--query "SELECT * FROM Spotify_Popularity" \
		> Spotify_Popularity.csv
```



### Manipulating data using SQL syntax

```shell
csvsql --query "SELECT * FROM Spotify_MusicAttributes LIMIT 1" \
	Spotify_MusicAttributes.csv	
	
csvsql --query "SELECT * FROM Spotify_MusicAttributes LIMIT 1" \
	data/Spotify_MusicAttributes.csv | csvlook
	
csvsql --query "SELECT * FROM Spotify_MusicAttributes LIMIT 1" \
	data/Spotify_MusicAttributes.csv > OneSongFile.csv
	
csvsql --query "SELECT * FROM file_a INNER JOIN file_b..." file_a.csv file_b.csv
```



### Pushing data back to database

```shell
csvsql --db "sqlite://SpotifyDatabase.db" \
		--insert Spotify_MusicAttributes.csv
		
		
csvsql --no-inference --no-constraints \
		--db "sqlite://SpotifyDatabase.db" \
		--insert Spotify_MusicAttributes.csv
```



### Python on command line

```shell
man python

# 파이썬 버전확인
python --version

# 파이썬 경로확인
which python

python
>>> print("hello world")
>>> exit()


echo "print('hello world')" > hello_world.py
```



### python package installation with pip

```shell
pip install --upgrade pip

pip list # 설치된 패키지 나열

pip install scikit-learn

pip install scikit-learn==0.19.2

pip install --upgrade scikit-learn

pip install scikit-learn statmodels

pip install --upgrade scikit-learn statmodels

cat requirements.txt
pip install -r requirements.txt
```



### Data job automation with cron

```shell
 crontab -l # 예약되어있는 작업을 출력
 
 man crontab
 
 echo "* * * * * python create_model.py" | crontab
```



