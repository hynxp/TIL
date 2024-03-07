# 브라우저의 전역 변수 충돌 문제를 회피하는 방법

<br>

```javascript
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });

        $('#btn-update').on('click', function () {
            _this.update();
        });

        $('#btn-delete').on('click', function () {
            _this.delete();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    update : function() {
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };

        var id = $('#id').val();

        $.ajax({
            type: 'PUT',
            url: '/api/v1/posts/' + id,
            dataType: 'json',
            contentType: 'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function(){
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function() {
            alert(JSON.stringify(error));
        });
    }
};

main.init();
```
<br>

**첫문장에 `var main = { .. }` 라는 변수의 속성으로 function을 추가한 이유가 뭘까?**

만약 같은 화면에서 **또다른 a.js**가 추가되어 init, save, update function()이 있다면

브라우저의 스코프는 공용 공간으로 쓰이기 때문에 **나중에 로딩된 js의 function들이 먼저 로딩된 js의 function을 덮어쓰게 된다.**

여러 사람이 참여하는 프로젝트에서는 중복된 함수명이 있을 확률이 높으므로 index.js만의 유효범위(scope)를 만들어 사용하는 것이다.