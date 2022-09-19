.helmignore 파일은 helm chart에 포함하고 싶지 않은 파일을 명시하는 데 사용할 수 있다.

.helmignore 파일이 존재하면 helm package 명령어는 해당 파일에 명시된 패턴과 매칭되는 파일을 패키징하는 데 제외한다.

.helmignore 파일은 Unix shell glob 매칭, 상대 경로 매칭, 부정(! 접두사)를 지원한다. 한 줄에 하나의 패턴만 고려된다.

아래는 예시다:

``` yaml
# comment

# Match any file or path named .git
.git

# Match any text file
*.txt

# Match only directories named mydir
mydir/

# Match only text files in the top-level directory
/*.txt

# Match only the file foo.txt in the top-level directory
/foo.txt

# Match any file named ab.txt, ac.txt, or ad.txt
a[b-d].txt

# Match any file under subdir matching temp*
*/temp*

*/*/temp*
temp?
```

.gitignore와 차이점은 다음과 같다:

- '**' 문법은 지원하지 않는다.
- globbing library는 Go 언어의 'filepath.Match'이며, fnmatch(3)가 아니다.
- Trailing spaces are always ignored (there is no supported escape sequence)
- There is no support for '!' as a special leading sequence.