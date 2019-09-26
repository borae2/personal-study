# Cookie SameSite
* 쿠키의 Header 설정 중 SameSite 에 관한 내용


| 값 | 설명 | 비고 |
|---|---|---|
| Strict | 브라우저 상의 URL과 매칭 될 때에만 보내진다. | |
| Lax | 다른 주소에 있다가 다시 쿠키를 발급한 주소로 돌아올 떄에 보내진다. | |
| None | 다른 주소에서도 쿠키 유지하도록 설정 | |

### 예시)
* 프로모션 정보 (2달짜리) 는, 해당 사이트에서는 유지가 되어야함.
* 그렇다고 다른 사이트에도 쿠키가 보내지면, request에 overhead만 주게 됨.
* 세션과 같은 정보는 CSRF 공격을 피하기 위해서라도 다른 사이트로 보내지지 않아야 하지만,
* 프로모 정보는 상관이 없긴 하므로, Strict 일 필요가 없다. Lax를 사용하면 유지도 가능하고 다른 사이트로도 안보낸다.


## 버젼별 특징
### Chrome
* 80버젼에서는 SameSite=Lax 가 default (78은 베타, 76은 same-site-by-default-cookies flag를 enable 시)
* 76 이후부터는, SameSite=None 이면, secure flag가 붙어야함. 이는 https 요청만 허용된다는 것을 의미.
```
Set-Cookie: PHPSESSID=AB1234kjsdf9u2348djhd73; httpOnly; SameSite=None; secure;
```


### firefox
* 60 버젼부터 SameSite 쿠키 지원

## 결론
* 웍스는 현재 SameSite 설정 하지 않고있음.
* default는 ???
* domain 경로로 관리하고있음
* isRefreshed 활용?


# 출처 및 참고할 reference
* https://www.chromestatus.com/feature/5088147346030592
* https://web.dev/samesite-cookies-explained
* https://blog.mozilla.org/security/2018/04/24/same-site-cookies-in-firefox-60/
* https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
* https://www.netsparker.com/blog/web-security/same-site-cookies-by-default/
* https://www.chromestatus.com/feature/5633521622188032
* https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-02#section-5.3.7
* https://www.owasp.org/index.php/SameSite (default value?)
