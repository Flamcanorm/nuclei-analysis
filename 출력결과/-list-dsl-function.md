---
유형: 실행 결과
상태: false
명령: ./nuclei -list-dsl-function
상세: 현재 Nuclei에 등록된 DSL 함수 서명 출력
---

# `-list-dsl-function` 실행 결과 #전지성

## 입력

```bash
./nuclei -list-dsl-function
```

짧은 별칭인 `./nuclei -ldf`도 같은 결과를 출력한다.

## 출력

```text
[INF] The available custom DSL functions are:
        aes_cbc(arg1, arg2, arg3 interface{}) interface{}
        aes_gcm(arg1, arg2 interface{}) interface{}
        base64(arg1 interface{}) interface{}
        base64_decode(arg1 interface{}) interface{}
        base64_py(arg1 interface{}) interface{}
        bin_to_dec(arg1 interface{}) interface{}
        compare_versions(firstVersion, constraints ...string) bool
        concat(args ...interface{}) string
        contains(arg1, arg2 interface{}) interface{}
        cookie_unsign(s string) string
        count(str, substr string) int
        date_time(dateTimeFormat string, optionalUnixTime interface{}) string
        dec_to_hex(arg1 interface{}) interface{}
        deflate(arg1 interface{}) interface{}
        ends_with(str string, suffix ...string) bool
        generate_dotnet_gadget(arg1, arg2, arg3, arg4 interface{}) interface{}
        generate_java_gadget(arg1, arg2, arg3 interface{}) interface{}
        generate_jwt(jsonString, algorithm, optionalSignature string, optionalMaxAgeUnix interface{}) string
        getNetworkPort(Port int, defaultPort int) int
        getNetworkPort(Port string, defaultPort string) string
        gzip(arg1 interface{}) interface{}
        gzip_decode(data string, optionalReadLimit int) string
        gzip_mtime(arg1 interface{}) interface{}
        hex_decode(arg1 interface{}) interface{}
        hex_encode(data interface{}) interface{}
        hex_to_dec(arg1 interface{}) interface{}
        hmac(arg1, arg2, arg3 interface{}) interface{}
        html_escape(s string, optionalConvertAllChars bool) string
        html_unescape(arg1 interface{}) interface{}
        index(arg1, arg2 interface{}) interface{}
        inflate(data string, optionalReadLimit int) string
        ip_format(arg1, arg2 interface{}) interface{}
        jarm(arg1 interface{}) interface{}
        join(separator string, elements ...interface{}) string
        json_minify(arg1 interface{}) interface{}
        json_prettify(arg1 interface{}) interface{}
        len(arg1 interface{}) interface{}
        line_ends_with(str string, suffix ...string) bool
        line_starts_with(str string, prefix ...string) bool
        llm_prompt(prompt string, optionalModel string) string
        md5(arg1 interface{}) interface{}
        mmh3(arg1 interface{}) interface{}
        oct_to_dec(arg1 interface{}) interface{}
        padding(arg1, arg2, arg3, arg4 interface{}) interface{}
        print_debug(args ...interface{})
        public_ip() string
        rand_base(length uint, optionalCharSet string) string
        rand_char(optionalCharSet string) string
        rand_int(optionalMin, optionalMax uint) int
        rand_ip(cidr ...string) string
        rand_text_alpha(length uint, optionalBadChars string) string
        rand_text_alphanumeric(length uint, optionalBadChars string) string
        rand_text_numeric(length uint, optionalBadNumbers string) string
        regex(arg1, arg2 interface{}) interface{}
        regex_all(pattern string, inputs ...string) bool
        regex_any(pattern string, inputs ...string) bool
        remove_bad_chars(arg1, arg2 interface{}) interface{}
        repeat(arg1, arg2 interface{}) interface{}
        replace(arg1, arg2, arg3 interface{}) interface{}
        replace_regex(arg1, arg2, arg3 interface{}) interface{}
        resolve(format string) string
        resolve(host string) string
        reverse(arg1 interface{}) interface{}
        rsa_encrypt(arg1, arg2 interface{}) interface{}
        sha1(arg1 interface{}) interface{}
        sha256(arg1 interface{}) interface{}
        sha512(arg1 interface{}) interface{}
        sort(elements ...interface{})
        sort(input number) string
        sort(input string) string
        split(input string, n int)
        split(input string, separator string, optionalChunkSize)
        starts_with(str string, prefix ...string) bool
        substr(str string, start int, optionalEnd int)
        to_bool(arg1 interface{}) interface{}
        to_lower(arg1 interface{}) interface{}
        to_number(arg1 interface{}) interface{}
        to_string(arg1 interface{}) interface{}
        to_title(s, optionalLang string) string
        to_unix_time(input string, optionalLayout string) int64
        to_upper(arg1 interface{}) interface{}
        trim(arg1, arg2 interface{}) interface{}
        trim_left(arg1, arg2 interface{}) interface{}
        trim_prefix(arg1, arg2 interface{}) interface{}
        trim_right(arg1, arg2 interface{}) interface{}
        trim_space(arg1 interface{}) interface{}
        trim_suffix(arg1, arg2 interface{}) interface{}
        uniq(elements ...interface{})
        uniq(input number) string
        uniq(input string) string
        unix_time(optionalSeconds uint) float64
        unpack(arg1, arg2 interface{}) interface{}
        url_decode(arg1 interface{}) interface{}
        url_encode(s string, optionalEncodeAllSpecialChars bool) string
        wait_for(seconds uint)
        wappalyzer(headers, body string)
        xor(args ...interface{}) interface{}
        zlib(arg1 interface{}) interface{}
        zlib_decode(data string, optionalReadLimit int) string
        contains_all(body interface{}, substrs ...string) bool
        contains_any(body interface{}, substrs ...string) bool
        equals_any(s interface{}, subs ...interface{}) bool
        hex_encode(data interface{}, optionalFormat string) interface{}
        join(separator string, elements []interface{}) string
        zip(file_entry string, content string, ... ) []byte
```

## 이 결과를 읽는 방법

- 첫 단어는 DSL 함수 이름이다.
- 괄호 안은 함수에 전달할 값과 값의 종류다.
- 괄호 뒤의 마지막 타입은 함수가 돌려주는 결과의 종류다.
- 같은 이름이 여러 번 나오면 받을 수 있는 값의 형태가 여러 가지라는 뜻이다.
- 이 목록은 현재 실행한 Nuclei와 DSL 패키지 버전에 등록된 함수 기준이다.

## 연결 문서

- 기능과 코드 흐름: [-list-dsl-function 처음부터 이해하기](<../설명서/14_-list-dsl-function 상세.md>)
- `main.go`의 실행 위치: [main.md L74](<../01_Main_Flow/main.md>)
- 모든 결과 목록: [결과 폴더 안내](<README.md>)
