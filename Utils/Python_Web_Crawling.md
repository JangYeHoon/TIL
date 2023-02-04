## Python Web Crawling

해당 글은 재미로 만들었던 웹 크롤링 코드입니다. 사이트들의 핫딜 품목을 크롤링하여 찾고 있는 단어를 포함한 제품이 있는지 확인하는 코드입니다.

크롤링 사이트 목록

- 퀘이사존
- 뽐뿌
- 에펨코리아

위 3개의 사이트에서 핫딜게시판의 최신 항목을 가져와 이름을 크롤링하여 제가 설정한 이름이 포함되어 있으면 출력해주는 코드입니다.

```python
# 참고 블로그 : https://velog.io/@changhtun1/%ED%8C%8C%EC%9D%B4%EC%8D%AC%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%9B%B9-%ED%81%AC%EB%A1%A4%EB%A7%81

from urllib.request import Request
from urllib.request import urlopen
from bs4 import BeautifulSoup

def find_product_in_webpage(url, tag, tag_class=None):
    find_product = []
    req = Request(url, headers={'User-Agent':'Mozila/5.0'})
    webpage = urlopen(req)
    parse = BeautifulSoup(webpage, features="html.parser")

    objects = parse.find_all(tag, tag_class)
    for i in objects:
        if any(keyword.lower() in str(i).lower() for keyword in keywords):
                find_product.append(i)

    return find_product


keywords = ['네파', '마우스', '귀뚜라미', 'razer']

for i in find_product_in_webpage("https://www.fmkorea.com/hotdeal", 'h3'):
    print(i)

for i in find_product_in_webpage("https://www.ppomppu.co.kr/zboard/zboard.php?id=ppomppu", 'font', "list_title"):
    print(i)

for i in find_product_in_webpage("https://quasarzone.com/bbs/qb_saleinfo", 'span', "ellipsis-with-reply-cnt"):
    print(i)
```



