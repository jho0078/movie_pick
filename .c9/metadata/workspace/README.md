{"changed":false,"filter":false,"title":"README.md","tooltip":"/README.md","value":"\n\n## 대전 2반 1학기 프로젝트 보고서 - 빨간차(안상현, 정세민)\n\n\n\n\n\n### 1. 완성된 소스코드의 Gitlab 주소\n\n```\n안상현 : https://lab.ssafy.com/ash92kr/final_project\n정세민 : https://lab.ssafy.com/jho0078/project_final\n```\n\n\n\n\n\n### 2. 팀원 정보 및 업무 분담 내역\n\n```\n안상현 : 웹크롤링 및 데이터 수정, DB 설계, 프론트엔드(특히 댓글 CRUD 작성), 보고서 및 PPT 작성\n정세민 : DB migration, 프론트엔드(특히 템플릿 설정, account 작성, 추천 알고리즘 작성), 발표\n```\n\n\n\n\n\n### 3. 목표 서비스 구현 및 실제 구현 정도\n\n\n\n전반적으로 필수 기능을 우선적으로 구현하는 데 목표를 두었다\n\n##### (1) 관리자 권한의 유저만 화 등록 및 수정 / 삭제 권한을 가진다(별도의 관리자 권한 뷰가 구성되어야 함)\n\n##### (2) 관리자 권한의 유저는 유저 관리 (수정 / 삭제 권한)을 가집니다. \n\nDjango에서는 기본적으로 models.py를 설정하고 python manage.py createsuperuser를 입력하면 자동으로 admin 권한이 생성된다.\n\n```python\n# movies - admin.py\nfrom django.contrib import admin\nfrom .models import Movie, Actor, Genre, Comment\n\nclass MovieAdmin(admin.ModelAdmin):\n    list_display =['actor1', 'poster_url', 'description', 'genre_id',] \n\nadmin.site.register(Movie, MovieAdmin)\n    \nclass ActorAdmin(admin.ModelAdmin):\n    list_display =['actor_id', 'name',]\n                    \nadmin.site.register(Actor, ActorAdmin)\n\nclass GenreAdmin(admin.ModelAdmin):\n    list_display =['name',]\n    \nadmin.site.register(Genre, GenreAdmin)\n\nclass CommentAdmin(admin.ModelAdmin):\n    list_display = ['user_id', 'movie_id', 'content', 'score']\n    \nadmin.site.register(Comment, CommentAdmin)\n```\n\n\n\n-> 여기 캡처화면 넣기\n\n\n\n##### (3) 모든 로그인 된 유저는 영화에 대한 평점을 등록 / 수정 / 삭제 등을 할 수 있어야 한다\n\n```python\n# movies - views.py\nfrom django.shortcuts import render, redirect, get_object_or_404, get_list_or_404\nfrom django.contrib.auth.decorators import login_required\nfrom django.views.decorators.http import require_POST\nfrom .models import Movie, Actor, Genre, Comment\nfrom .forms import CommentForm\n\n@require_POST\n@login_required    \ndef comment_create(request, movie_pk):\n    movie = get_object_or_404(Movie, pk=movie_pk)  \n    form = CommentForm(request.POST)\n    if form.is_valid():\n        comment = form.save(commit=False)\n        comment.user_id = request.user.id\n        comment.movie_id = movie_pk\n        # comment.movie = movie_pk\n        comment.save()\n    return redirect('movies:detail', movie_pk)\n\n\n@require_POST\n@login_required\ndef comment_delete(request, movie_pk, comment_pk):\n    comment = get_object_or_404(Comment, pk=comment_pk)\n    comment.delete()\n    return redirect('movies:detail', movie_pk)\n    \n\n@login_required\ndef comment_update(request, movie_pk, comment_pk):\n    comment = get_object_or_404(Comment, pk=comment_pk)\n    \n    if comment.user == request.user:\n        if request.method == \"POST\":\n            form = CommentForm(request.POST, instance=comment)\n            if form.is_valid():\n                movie = form.save()\n                return redirect('movies:detail', movie_pk)\n        else:\n            form = CommentForm(instance=comment)\n    else:\n        return redirect('movies:detail', movie_pk)\n    context = {\n        'form': form,\n        'comment': comment,\n    }\n    return render(request, 'movies/detail.html', context)\n```\n\n\n\n```python\n# movies - urls.py\n    path('<int:movie_pk>/comments/<int:comment_pk>/delete/', views.comment_delete, name='comment_delete'),\n    path('<int:movie_pk>/comments/<int:comment_pk>/update/', views.comment_update, name='comment_update'),\n    path('<int:movie_pk>/comments/', views.comment_create, name='comment_create'),\n```\n\n\n\n```html\n<!--movies - detail.html-->\n{% load static %}\n\n{% load crispy_forms_tags %}\n\n<!DOCTYPE html>\n<html lang=\"en\">\n\n<head>\n  <meta charset=\"utf-8\">\n  <meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">\n\n...\n\n      <!--댓글 작성-->\n      <form action=\"{% url 'movies:comment_create' movie.pk %}\" method=\"POST\">\n        {% csrf_token %}\n        {{ form|crispy }}\n        <input type=\"submit\" value=\"입력\"/>\n      </form>\n      \n      <hr>\n      <br>\n\n      <!--댓글 조회-->\n      <h2>입력한 댓글들</h2>\n      \n      {% for comment in movie.comment_set.all %}\n        <!--댓글 수정 및 삭제-->\n        {% if request.user == comment.user %}   <!--댓글 단 사람만 수정 삭제 가능-->\n          \n        유저 : {{ comment.user }}<br>\n        <form action=\"{% url 'movies:comment_update' movie.pk comment.pk %}\" method=\"POST\" style=\"display:inline;\">\n          {% csrf_token %}\n          <input type=\"number\" name=\"score\" value=\"{{ comment.score }}\"/>\n          <input type=\"text\" name=\"content\" value=\"{{ comment.content }}\"/>\n          <input type=\"submit\" value=\"수정\"/>\n        </form>\n          <a href=\"{% url 'movies:comment_delete' movie.pk comment.pk %}\">[삭제]</a>\n        <hr>\n\n        {% else %}\n        \n        <p>유저 : {{ comment.user }}<br>\n           평점 : {{ comment.score }}<br>\n           내용 : {{ comment.content }}</p>\n        <hr>\n        \n        {% endif %}\n    \n      {% endfor %}\n```\n\n\n\n\n\n##### (4) 평점을 등록한 유저는 해당 정보를 기반으로 영화를 추천 받을 수 있어야 한다\n\n```python\n# accounts - forms.py\nfrom django import forms\nfrom django.contrib.auth import get_user_model\nfrom django.contrib.auth.forms import UserCreationForm, UserChangeForm\nfrom .models import Profile\n\nclass CustomUserCreationForm(UserCreationForm):\n    class Meta(UserCreationForm.Meta):    # Meta class 도 상속받을 수 있다.\n        model = get_user_model()\n        fields = UserCreationForm.Meta.fields\n        \nclass ProfileForm(forms.ModelForm):\n    class Meta:\n        model = Profile\n        fields = ['nickname','introduction','favorite_genre']\n        \n        FAVORITE_GENRE_CHOICE = (\n            ('가족','가족'),('공포(호러)','공포(호러)'),('다큐멘터리','다큐멘터리'),('드라마','드라마'),\n            ('멜로/로맨스','멜로/로맨스'),('뮤지컬','뮤지컬'),('미스터리','미스터리'),('범죄','범죄'),\n            ('사극','사극'),('서부극(웨스턴)','서부극(웨스턴)'),('스릴러','스릴러'),('애니메이션','애니메이션'),('액션','액션'),\n            ('어드벤처','어드벤처'),('전쟁','전쟁'),('코미디','코미디'),('판타지','판타지'),('SF','SF')\n            )\n            \n        widgets = {\n            'introduction': forms.Textarea,\n            'favorite_genre': forms.Select(choices=FAVORITE_GENRE_CHOICE),\n        }\n```\n\n\n\n```html\n<!--accounts - profile_update.html-->\n    <div class=\"container\">\n        <form action=\"{% url 'accounts:profile_update'%}\" method='POST'>\n            {% csrf_token %}\n            {% bootstrap_form profile_form %}\n            <button type=\"submit\" class=\"btn btn-warning\">BACK</button>\n            <button type=\"submit\" class=\"btn btn-primary\">수정</button>\n            <!--<input type=\"submit\" value=\"Submit\"/>-->\n        </form>\n    </div>\n    </div>\n```\n\n\n\n```python\n# movies - views.py\ndef list(request):\n    current_movies = Movie.objects.order_by('-year')[:8]\n    if request.user.is_authenticated and request.user.profile.favorite_genre:\n        genre = Genre.objects.filter(name=request.user.profile.favorite_genre)[0]\n        user_genre_movies = Movie.objects.filter(genre_id=genre.id).order_by('-user_rating')[:8]\n        context = {\n            'current_movies': current_movies,\n            'user_genre_movies': user_genre_movies,\n        }\n    else:\n        context = {\n            'current_movies': current_movies,\n        }\n    return render(request, 'movies/list.html', context)\n```\n\n\n\n```html\n<!--movies - list.html-->\n<!--유저가 좋아하는 장르 영화 8개 출력-->\n      {% if user.is_authenticated and user.profile.favorite_genre %}\n        <h2 class=\"text-center mt-5\">{{ user.username }}가 좋아하는 장르 추천 영화</h2>\n        <div class=\"row tm-albums-container grid\">\n          {% for movie in user_genre_movies %}\n          <div class=\"col-sm-6 col-12 col-md-6 col-lg-3 col-xl-3 tm-album-col\">\n            <a href=\"{% url 'movies:detail' movie.pk %}\">\n              <figure class=\"effect-sadie\">\n                <img src=\"{{ movie.poster_url }}\" alt=\"Image\" class=\"img-fluid\">\n                  <figcaption>\n                    <a href=\"{% url 'movies:detail' movie.pk %}\"><h2>{{ movie.movie_name }}</h2></a>\n                    <p>평점 : {{ movie.user_rating }} / 장르 : {{ movie.genre }}</p>\n                  </figcaption>\n              </figure>\n            </a>\n          </div>\n          {% endfor %}\n        </div>\n        <hr>\n      {% elif not user.is_authenticated %}\n        <h2 class=\"text-center mt-5\">로그인 후 좋아하는 장르 영화를 추천받으세요.</h2>\n      {% else %}\n        <h2 class=\"text-center mt-5\">장르설정 후 좋아하는 장르 영화를 추천받으세요.</h2>\n      {% endif %}\n```\n\n\n\n우리 팀은 유저가 가장 좋아한다고 선택한 장르의 영화를 추천해주는 기능을 구현했다.\n\n선택한 장르의 영화 8개를 보여주되, 평점이 가장 높은 순서대로 정렬해서 로그인 시 첫 화면에 보여준다\n\n\n\n##### (5) 데이터베이스에 기록되는 모든 정보는 유효성 검사를 진행해야 하며, 필요에 따라 해당하는 정보를 클라이언트 화면에 띄워줄 수 있어야 한다\n\n#####  (6) HTTP method와 상태 코드는 적절하게 반환되어야 하며, 필요에 따라 해당하는 에러 페이지도 구성을 할 수 있다\n\n```python\n# movies - views.py\nfrom django.shortcuts import render, redirect, get_object_or_404, get_list_or_404\nfrom django.contrib.auth.decorators import login_required\nfrom django.views.decorators.http import require_POST\nfrom .models import Movie, Actor, Genre, Comment\nfrom .forms import CommentForm\n\ndef detail(request, movie_pk):\n    movie = get_object_or_404(Movie, pk=movie_pk)\n    actors = get_list_or_404(Actor.objects.order_by('-pk'))\n    form = CommentForm(request.POST)\n    context = {\n        'movie': movie,\n        'actors': actors,\n        'form': form,\n    }\n    return render(request, 'movies/detail.html', context)\n\n@require_POST\n@login_required    \ndef comment_create(request, movie_pk):\n    movie = get_object_or_404(Movie, pk=movie_pk)  \n    form = CommentForm(request.POST)\n    if form.is_valid():\n        comment = form.save(commit=False)\n        comment.user_id = request.user.id\n        comment.movie_id = movie_pk\n        comment.save()\n    return redirect('movies:detail', movie_pk)\n\n@require_POST\n@login_required\ndef comment_delete(request, movie_pk, comment_pk):\n    comment = get_object_or_404(Comment, pk=comment_pk)\n    comment.delete()\n    return redirect('movies:detail', movie_pk)\n    \n\n@login_required\ndef comment_update(request, movie_pk, comment_pk):\n    comment = get_object_or_404(Comment, pk=comment_pk)\n    \n    if comment.user == request.user:\n        if request.method == \"POST\":\n            form = CommentForm(request.POST, instance=comment)\n            if form.is_valid():\n                movie = form.save()\n                return redirect('movies:detail', movie_pk)\n        else:\n            form = CommentForm(instance=comment)\n    else:\n        return redirect('movies:detail', movie_pk)\n    context = {\n        'form': form,\n        'comment': comment,\n    }\n    return render(request, 'movies/detail.html', context)\n\n\n@login_required\ndef like(request, movie_pk):\n    movie = get_object_or_404(Movie, pk=movie_pk)\n    if request.user in movie.like_users.all():\n        movie.like_users.remove(request.user)\n        liked = False\n    else:\n        movie.like_users.add(request.user)\n        liked = True\n    # return redirect('posts:list')\n    context = {\n        'liked': liked,\n        'count': movie.like_users.count(),\n    }\n    return JsonResponse(context)\n    \n@login_required\ndef comment_edit(request, movie_pk, comment_pk):\n    movie = get_object_or_404(Movie, pk=movie_pk)\n    comment = get_object_or_404(Comment, pk=comment_pk)\n    context = {\n        'movie': movie,\n        'comment': comment,\n    }\n    return render(request, 'movies/contact.html', context)\n```\n\n\n\n```python\n# accounts - views.py\nfrom django.shortcuts import render, redirect, get_object_or_404\nfrom django.contrib.auth.forms import AuthenticationForm, PasswordChangeForm\n\n...\n\ndef signup(request):\n    if request.user.is_authenticated:\n        return redirect('movies:list')\n        \n    if request.method == 'POST':\n        signup_form = CustomUserCreationForm(request.POST)\n        if signup_form.is_valid():\n            signup_user = signup_form.save()\n            Profile.objects.create(user=signup_user)\n            auth_login(request, signup_user)\n            return redirect('movies:list')\n    else:\n        signup_form = CustomUserCreationForm()\n    context = {'signup_form': signup_form}\n    return render(request, 'accounts/signup.html', context)\n    \ndef login(request):\n    if request.user.is_authenticated:\n        return redirect('movies:list')\n    \n    if request.method == 'POST':\n        login_form = AuthenticationForm(request, request.POST)\n        if login_form.is_valid():\n            login_user = login_form.get_user()\n            auth_login(request, login_user)\n            return redirect(request.GET.get('next') or 'movies:list')\n    else:\n        login_form = AuthenticationForm()\n    context = {'login_form': login_form}\n    return render(request, 'accounts/login.html', context)\n\ndef logout(request):\n    auth_logout(request)\n    return redirect('movies:list')\n    \ndef people(request, username):\n    people = get_object_or_404(get_user_model(), username=username)\n    context = {'people': people,}\n    return render(request, 'accounts/people.html', context)\n    \ndef delete(request):\n    request.user.delete()\n    return redirect('posts:list')\n    \n@login_required\ndef profile_update(request):\n    profile = Profile.objects.get_or_create(user=request.user)\n    if request.method == 'POST':\n        profile_form = ProfileForm(data=request.POST, instance=request.user.profile)\n        if profile_form.is_valid():\n            profile_form.save()\n            return redirect('people', request.user.username)\n    else:\n        profile_form = ProfileForm(instance=request.user.profile)\n    context = {'profile_form': profile_form}\n    return render(request, 'accounts/profile_update.html', context)\n\n@login_required\ndef follow(request, user_pk):\n    people = get_object_or_404(get_user_model(), pk=user_pk)\n    # people 을 팔로워하고 있는 모든 유저에 현재 접속 유저가 있다면\n    if request.user in people.followers.all():\n    # 언팔로우\n        people.followers.remove(request.user)\n    # 아니면\n    else:\n    # 팔로우\n        people.followers.add(request.user)\n    return redirect('people', people.username)\n```\n\n\n\n5번의 경우, movies와 accounts에 forms.py를 만들어 폼을 완성했다. \n\n이후 views.py에서 해당 폼을 거쳐 데이터를 입력, 수정, 삭제하는 경우(CUD), form.is_valid()를 통해 데이터가 누락되었는지, 해당 형식에 맞는지 등을 확인했다.\n\n이러한 유효성 검사를 통과한 다음에야 form.save()를 통해 데이터를 DB에 저장했다.\n\n또한, 수정(Update)은 기존에 유저가 입력한 내용을 보여주어야 한다.\n\n이에 CommentForm(request.POST, instance=comment)과 같이 기존에 입력한 내용을 instance에 담아서 보여주도록 했다.\n\n\n\n6번의 경우 get_object_or_404, get_list_or_404 라이브러리를 import해서 template에서 movie, comment 객체를 불러올 수 없는 경우 404 에러를 보여주도록 나타냈다.\n\n\n\n##### (7) 그 외 추가적인 기능\n\n\n\n###### 1) 검색 - 이용자가 키워드를 입력하면 키워드와 일치하는 영화 제목을 보여준다\n\n```python\n# movies - views.py\n\ndef movie_list(request):\n    movies = Movie.objects.all()\n    keyword = request.GET.get('keyword', '') # GET request의 인자중에 q 값이 있으면 가져오고, 없으면 빈 문자열 넣기\n    if keyword: # q가 있으면\n        movies = movies.filter(movie_name__icontains=keyword) # 제목에 q가 포함되어 있는 레코드만 필터링\n    context = {\n        'movies': movies\n    }\n    return render(request, 'movies/movie_list.html', context)\n```\n\n\n\n```python\n# movies - urls.py\n\tpath('movielist/', views.movie_list, name='movie_list'),\n```\n\n\n\n```html\n<!--movies - list.html, detail.html-->\n   <div class=\"container\">\n      <div class=\"tm-search-form-container\">\n        <form action=\"{% url 'movies:movie_list' %}\" method=\"GET\" class=\"form-inline tm-search-form\">\n          <div class=\"text-uppercase tm-new-release\"><h2>영화 검색</h2></div>\n          <div class=\"form-group tm-search-box\">\n            <input type=\"text\" name=\"keyword\" class=\"form-control tm-search-input\" placeholder=\"Type your keyword ...\">\n            <input type=\"submit\" value=\"Search\" class=\"form-control tm-search-submit\">\n          </div>\n        </form>\n      </div>\n```\n\n\n\n###### 2) 좋아요 - 유저가 특정 영화에 좋아요 버튼을 누르면 profile 페이지에서 그 영화의 이미지가 나옴\n\n```python\n# movies - views.py\n\nfrom django.shortcuts import render, redirect, get_object_or_404, get_list_or_404\nfrom django.contrib.auth.decorators import login_required\nfrom .models import Movie, Actor, Genre, Comment\nfrom django.http import JsonResponse\n\n@login_required\ndef like(request, movie_pk):\n    movie = get_object_or_404(Movie, pk=movie_pk)\n    if request.user in movie.like_users.all():\n        movie.like_users.remove(request.user)\n        liked = False\n    else:\n        movie.like_users.add(request.user)\n        liked = True\n    context = {\n        'liked': liked,\n        'count': movie.like_users.count(),\n    }\n    return JsonResponse(context)\n```\n\n\n\n```python\n# movies - urls.py\n\tpath('<int:movie_pk>/like/', views.like, name='like'),\n```\n\n\n\n```html\n<!--movies - detail.html-->\n\n            <i class=\"{% if user in movie.like_users.all %}fas{% else %}far{% endif %} fa-heart fa-lg like-button\" data-id=\"{{ movie.pk }}\" style=\"color:crimson; display:inline;\"></i>\n            <span id=\"like-count-{{ movie.pk }}\" class=\"card-text\" style=\"display:inline;\">{{ movie.like_user.count }}</span>명이 이 영화를 찜했습니다.\n            <p>{{ movie.like_user.count }}명이 이 영화를 찜했습니다.</p>\n            {{ movie.like_user.count }}\n...\n\n  <script>\n    // 좋아요 버튼들을 모두 선택\n    const likeButtons = document.querySelectorAll('.like-button')\n    likeButtons.forEach(button => {\n        // 각각의 버튼에 클릭 이벤트 설정\n        button.addEventListener('click', function (event) {\n            console.log(event)\n            // 좋아요 버튼의 해당 게시글 id\n            const movieId = event.target.dataset.id\n            // 좋아요 요청 전송\n            axios.get(`/movies/${movieId}/like/`)\n                .then( response => {\n                    document.querySelector(`#like-count-${movieId}`).innerText = response.data.count\n                    // 좋아요가 눌린상태인지 아닌지\n                    // 눌린 상태라면 class\n                    if (response.data.liked) {\n                        event.target.classList.remove('far')\n                        event.target.classList.add('fas')\n                    } else {\n                        event.target.classList.remove('fas')\n                        event.target.classList.add('far')\n                    }\n                })\n        })\n    })\n  </script>\n```\n\n\n\n\n\n### 4. 데이터베이스 모델링\n\n\n\n![모델링](https://user-images.githubusercontent.com/43332543/57835902-89446100-77fa-11e9-8706-8139a73776eb.JPG)\n\n\n\n\n\n### 5. 핵심 기능\n\n\n\n(1) 검색 기능 : 이용자가 원하는 키워드를 입력하면 해당 키워드를 가진 영화명이 나온다\n\n이후, 포스터 이미지를 클릭하면 해당 영화에 대한 상세 정보를 제공함\n\n\n\n(2) 정보 제공 기능 : 영화 이미지를 클릭하면 웹크롤링한 정보가 모두 나온다\n\n특히, 영화진흥위원회에서 가져온 영화 기본 정보, 네이버 영화 API에서 가져온 포스터 및 전체 평점, 유투브에서 가져온 동영상 링크 등을 한 자리에서 볼 수 있다\n\n\n\n\n\n### 6. 배포 서버 URL\n\n```\nhttp://movies-dev.ap-northeast-2.elasticbeanstalk.com\n```\n\n\n\n\n\n### 7. 느낀 점\n\n```\n(1) 안상현 : 이전 프로젝트보다 처음부터 끝까지 모두 실행한다는 점에서 어려웠지만, 팀원의 도움으로 어려움을 극복할 수 있었다. 또한, 웹과 알고리즘을 결합시켜 하나의 결과물을 만들었다는 점에서 보람있었다. 개인 사정으로 하루 빠진 적이 있어서 다른 팀원이 보다 많은 양의 업무를 했다는 점에서 아쉬웠다. 이 자리를 빌어 고마움을 전한다.\n\n(2) 정세민 : api를 통해 데이터를 받아 db에 넣는 것 부터 장고를 이용한 프론트엔드 구현까지 약 4개월동안 배웠던 모든 것들을 사용해 하나의 영화추천 사이트를 만들었다는 점이 굉장히 기쁘다. 진행 과정에서 어려운 점이 많았지만 팀원과의 협력을 통해 잘 이겨나갔던 것 같다. 다만 디자인적으로 완성된 사이트를 만들지 못했다는 점에 아쉬움이 남는다.\n```\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n","undoManager":{"mark":-1,"position":-1,"stack":[]},"ace":{"folds":[],"scrolltop":120,"scrollleft":0,"selection":{"start":{"row":15,"column":0},"end":{"row":15,"column":0},"isBackwards":false},"options":{"guessTabSize":true,"useWrapMode":false,"wrapToView":true},"firstLineState":{"row":6,"state":"allowBlock","mode":"ace/mode/markdown"}},"timestamp":1557997079511}