# Community

#### 1. 게시글

* 게시글 전체 조회
  
  * 게시글 제목, 내용, 유저 확인 가능
  
  * 유저 클릭시 -> 유저의 my page로 이동
  
  ![c1](https://github.com/seuluv/Movie4resT/assets/121653143/8cfc8881-4d60-45f1-bb19-6b4eb325c64d)

* 게시글 작성
  
  ![c2](https://github.com/seuluv/Movie4resT/assets/121653143/f9c2fa36-5430-4ba1-8947-ebdcf14a7648)
  
  * Title 미입력시
  
  ![c3](https://github.com/seuluv/Movie4resT/assets/121653143/4c7a0950-3bef-4db4-9a63-a0b00d12ea65)
  
  * content 미입력시
  
  ![c4](https://github.com/seuluv/Movie4resT/assets/121653143/2852eb16-7458-45a2-90a1-9f20e596e450)

* 게시글 Detail
  
  * 게시글 작성자, 생성날짜, 수정날짜, 내용, 댓글 확인 가능
  
  ![c5](https://github.com/seuluv/Movie4resT/assets/121653143/26a0f98e-b25a-48d6-8f92-ecbda6b1b050)

* 게시글 수정, 삭제

    * 작성자와 로그인한 유저가 같을 경우 -> 수정, 삭제 버튼 활성화
    
    ![c6](https://github.com/seuluv/Movie4resT/assets/121653143/25e05d73-3a42-49b9-96be-2c296cd4127a)
  
   * 수정 시 기존 content를 Form에 채워줌
    
    ![c7](https://github.com/seuluv/Movie4resT/assets/121653143/89488e8c-89d9-45d1-ad43-9ffbe44f8702)


#### 서비스 대표 기능 설명 - 장르 블락

개요 : 사용자가 직접 선호장르 및 비선호 장르를 구분하고, 선호 장르만을 검색 결과 및 장르 추천에서 볼 수 있게 한다. 가령, 공포 및 스릴러 장르를 비선호하는 사용자는 검색 결과에서 해당 장르를 가진 영화를 볼 수 없다.

![g1](https://github.com/seuluv/Movie4resT/assets/121653143/80777867-e081-45f8-93e3-f577b916b547)

위 캡처는 로그인한 사용자 본인의 페이지에 접속했을 때만 출력되는 페이지로, 배경이 회색인 장르는 블락된 장르를 가리킨다.

![g2](https://github.com/seuluv/Movie4resT/assets/121653143/a12fe60d-2768-4d87-98c5-204305274d62)

블락장르가 없을 때  '히어로' 검색

![g3](https://github.com/seuluv/Movie4resT/assets/121653143/bc3cc110-af77-41d4-802d-3e4841183c3d)

'애니메이션'을 블락한 뒤 '히어로' 검색

### Django

```django
@api_view(['POST'])
def genre_block(request, genre_id):
    genre = get_object_or_404(Genre, pk=genre_id)

    if request.user.blocked_genres.filter(pk=genre_id).exists():
        request.user.blocked_genres.remove(genre)
    else:
        request.user.blocked_genres.add(genre)

    blocked_genres = request.user.blocked_genres.all()
    serializer = GenreSerializer(blocked_genres, many=True)
    return Response(serializer.data)
```

마이페이지에서 장르를 블락하고, 블락을 해제할 때마다 위 함수가 작동하여 데이터베이스를 관리한다.

```django
q_objects = Q()
blocked_genres = blocked_genres.split(',')
for genre in blocked_genres:
    q_objects |= Q(genre__name__icontains=genre)
    movies = Movie.objects.filter( Q(overview__icontains=movie_keyword) 
| qmt_objects).exclude(q_objects)
```

위 함수는 영화 검색 페이지로, 블락 장르 리스트가 문자열 형태로 전달되기 때문에, ' , '로 분리하여 전체 필터링할 때 제외하도록 한 것이다.



### 검색 기능 추가 설명

```django
for keyword in movie_keyword:
            qmt_objects &= Q(title__icontains=keyword) 
```

장르 블락에서 설명한 검색 기능과 연결하여, 입력받은 movie_keyword를 타이틀과 오버뷰 모두에서 검색하여 결과를 보여준다.

1. overview에 대해서는 keyword와 일치하는 한 '단어'가 있을 때에만 검색되도록 했다.

2.  타이틀에 대해서는 줄임말까지 검색되고, 띄어쓰기에 영향받지 않도록 하기 위하여 입력받은 movie_keyword의 한 글자 한 글자를 분리하여, 해당하는 글자가 모두 포함되어야 검색되도록 했다.

상기 1과 2 둘 중 하나만 충족되어도 검색 결과에 출력된다. 아래는 예시이다.

![g4](https://github.com/seuluv/Movie4resT/assets/121653143/9a793ba3-b5c8-43a9-b523-c1793bb3505a)

'존 윅' 검색

![g5](https://github.com/seuluv/Movie4resT/assets/121653143/d182de3e-440c-4aa6-ba85-23be288526c9)

'존윅' 검색

![g6](https://github.com/seuluv/Movie4resT/assets/121653143/1cf9cdae-f6d0-4aa0-9dab-30995e5e74bf)

 '쥬월' 검색

![g7](https://github.com/seuluv/Movie4resT/assets/121653143/33e4d0a9-360c-4774-895e-1a800ecfe3a8)

오버뷰에 포함된 배역 이름 '스타로드' 검색



영화 페이지의 장르별 영화 추천기능에 대해서는 [영화 추천 알고리즘 설명](README_recommendation.md) 에 기술하였다.
