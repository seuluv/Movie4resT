# 영화 추천 알고리즘

## 1. 스토리형 사용자 선호 장르 테스트

개요 : 숲의 초입에서 여행을 시작하여 영화관에 도달하는 스토리로 진행되는 지문 9개로 구성되어있다. 각 지문은 영화 장르에 대응되는 선택지 세 개로 구성된다. 가장 많이 선택된 장르를 가진 영화 중 하나가 랜덤으로 선택되어 사용자에게 제시된다.

### Django

```django
@api_view(['POST'])
def movie_test(request, choose_genre):
    if request.method == 'POST':
        movies = Movie.objects.filter(Q(genre__name__icontains=choose_genre))
        random_movie = random.choice(movies)
        serializer = MovieSerializer(random_movie)
        return Response(serializer.data)
```

POST로 전달 받은 최종 결과에 해당하는 영화를 랜덤으로 vue에 전달한다.

### vue

액션, 코미디, 드라마, 가족, 판타지, 공포, 로맨스, SF 스릴러 장르를 key로 하는 딕셔너리를 만들어, 각 선택지를 선택함에 따라 각 key의 value 값을 증가시킨다.

각 테스트 페이지를 자식 컴포넌트로, 최초에 모달을 실행하는 페이지를 부모 컴포넌트로 한다. 자식 컴포넌트에서 선택지를 클릭하면 선택지에 해당하는 장르가 emit으로 부모 컴포넌트로 전달되고, 부모 컴포넌트에서 상기한 딕셔너리의 값을 수정한다.

<img title="" src="https://github.com/seuluv/Movie4resT/assets/121653143/45c53004-0db6-4ce8-b7d1-a10bf9334201" alt="테스트플로우차트.png" width="425" data-align="center">

<img src="https://github.com/seuluv/Movie4resT/assets/121653143/3a15d985-c28b-4480-81ac-4628d7fa5ce6" title="" alt="" width="334"><img title="" src="README_recommendation_assets/2023-05-25-09-34-26-image.png" alt="" width="191">

```vue
this.choiceGenre[genrename] = this.choiceGenre[genrename] + 1
        const currentModalElement = document.getElementById(`TestModal${this.page - 1}`)
        const currentModal = bootstrap.Modal.getInstance(currentModalElement)
        if (currentModal) {
          currentModal.hide()
        }
```

하위 컴포넌트에서 emit을 발생시켜 상위 컴포넌트에서 실행된 이벤트 코드의 일부. 받은 장르의 value를 1 증가하고, 이전 모달 페이지를 숨긴다.

모달 페이지가 준비한 페이지 수 미만일 경우 다음 모달이, 이상일 경우 결과 모달이 나타나게 한다.

```vue
const choose_genre = Object.keys(this.choiceGenre).reduce((a, b) => this.choiceGenre[a] > this.choiceGenre[b] ? a : b)
        axios.post(`${API_URL}/movies/test/movie_test/${choose_genre}`)
        .then(response => {
        this.movie = response.data
      })
      .catch(err => {
          console.error(err)
        })
```

선택된 장르 중 최고값을 가지는 장르를 axios로 Django에 전달

---

## 2. 선호 장르 기반 추천

개요 : 인기순, 평점순 추천에 더불어 사용자가 설정한 선호 장르 중 랜덤 세 개의 장르를 뽑아 제시한다. 이 랜덤 장르는 페이지를 새로고침 할 때마다 바뀐다.

[주요기능 설명 - 장르 블락](README_main_func_genre_block.md) 의 장르 블락 기능을 바탕으로하며, [README](README.md)의 주요 기능 설명 - 장르별 페이지의 바탕이 된다.

### Django

```django
@api_view(['POST'])
def movie_list(request, genre):
    if request.method == 'POST':
        if genre == '최고':
            movies = Movie.objects.order_by('-vote_average')[:10]
        elif genre == '인기':
            movies = Movie.objects.order_by('-popularity')[:10]
        else:
            movies = list(Movie.objects.filter(Q(genre__name__icontains=genre)))
            random.shuffle(movies)

        serializer = MovieSerializer(movies, many=True)
        return Response(serializer.data)
```

django에서는 가진 데이터 중 평점과 인기가 높은 영화를 10개씩 받아 vue로 정보를 보낸다.

이 때, 앞 쪽 데이터만 노출되는 현상을 피하기 위해 장르 별로 추출할 때는 랜덤 셔플을 통해 골고루 출력될 수 있게 한다.



### vue

vue에서는 사용자가 제외한 장르 이외 중 랜덤으로 세 개를 선택하여 django로 전달하고, 상기한 django 코드에서 해당하는 장르만 필터링하여 출력한다.



