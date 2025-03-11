# 네이버 금융 크롤러

이 프로젝트는 네이버 금융 리서치 페이지에서 기업 분석 보고서를 크롤링하여 저장하는 Python 코드입니다. 웹 스크래핑을 통해 종목별 리포트와 코멘트를 수집하여 JSON 형식으로 저장합니다.

## 기능 설명

### 1. `link_comment(url)`
특정 기업 분석 리포트 페이지에서 코멘트를 추출하는 함수입니다.
- HTTP 요청을 보낼 때 `Session`을 사용하여 쿠키를 유지합니다.
- `429 Too Many Requests` 오류가 발생하면 일정 시간 랜덤 딜레이 후 재시도합니다.
- `BeautifulSoup`을 이용해 `em` 태그에서 코멘트를 찾습니다.

```python
def link_comment(url):
    try:
        # 세션 생성 (쿠키 유지)
        session = requests.Session()
        session.headers.update({
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36"
        })

        # HTML 가져오기 (429 방지)
        for _ in range(3):  # 최대 3번 재시도
            response = session.get(url)
            if response.status_code == 429:
                print("⚠ 429 Too Many Requests - 재시도 중...")
                time.sleep(random.uniform(3, 7))  # 3~7초 랜덤 딜레이
            else:
                break  # 정상 응답이면 루프 탈출

        response.raise_for_status()  # HTTP 오류 확인

        # BeautifulSoup으로 HTML 파싱
        soup = BeautifulSoup(response.content, "html.parser")

        # 특정 <em> 태그 찾기
        comments = soup.find_all('em', class_='coment')

        if comments:
            return [comment.text.strip() for comment in comments]  # 리스트로 반환
        else:
            print("❌ 해당 클래스를 가진 태그를 찾을 수 없습니다.")
            return []

    except requests.exceptions.RequestException as e:
        print(f"❌ 웹사이트 요청 중 오류 발생: {e}")
    except Exception as e:
        print(f"❌ 기타 오류 발생: {e}")
```

### 2. `news_list(url)`
주어진 URL에서 기업별 리포트를 수집하는 함수입니다.
- `requests.Session`을 사용하여 HTTP 요청을 보냅니다.
- `tr` 태그 내에서 종목명, 리포트 링크, 날짜 정보를 추출합니다.
- 추출된 링크에서 `link_comment()`를 호출하여 코멘트도 함께 가져옵니다.

```python
def news_list(url):
```

### 3. `url_generator(n)`
네이버 금융 리서치 페이지의 여러 페이지 링크를 생성하는 함수입니다.
- 기본 URL (`https://finance.naver.com/research/company_list.naver?&page=`)을 기반으로 원하는 페이지 수(`n`)만큼 링크를 생성합니다.

```python
def url_generator(n):
```

### 4. `update_stock(url)`
크롤링한 데이터를 JSON 파일(`stock_data.json`)에 저장하는 함수입니다.
- 기존 데이터를 로드하여 새로운 데이터를 추가합니다.
- 새로운 종목이 있으면 딕셔너리에 추가한 후 JSON 파일로 저장합니다.

```python
def update_stock(url):
```

### 5. 메인 실행 코드
크롤러가 100페이지까지 탐색하며 데이터를 수집합니다.

```python
for link in url_generator(100):
    update_stock(link)
```

## 실행 방법

1. `requests`와 `BeautifulSoup4` 패키지가 필요합니다. 설치가 되어 있지 않다면 아래 명령어로 설치할 수 있습니다.
   ```bash
   pip install requests beautifulsoup4
   ```
2. 코드를 실행하면 `stock_data.json` 파일이 생성되며 크롤링한 데이터가 저장됩니다.

## 주의 사항
- 네이버의 `robots.txt` 정책을 준수하며 크롤링할 것.
- `429 Too Many Requests` 오류를 방지하기 위해 적절한 딜레이를 추가했지만, IP가 차단될 가능성이 있음.
- JSON 데이터 저장 경로를 적절히 설정해야 합니다.

---

이 프로젝트는 네이버 금융 데이터를 분석하거나 기업 리포트를 수집하여 감성 분석 등에 활용할 수 있습니다. 필요에 따라 코드를 수정하여 활용하면 좋습니다.
