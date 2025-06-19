### ⚠️ uproject를 열려고 할때 아래 문구가 뜨며 열리지 않음
```
The following modules are missing or built with a different engine version:

  ProjectName

Would you like to rebuild them now?
```

- [해결방법](https://designerd.tistory.com/entry/UE-The-following-modules-are-missing-or-built-with-a-different-engine-version-Would-you-like-to-rebuild-them-now)
  1. Visual Studio Installer를 확인해서 업데이트하기
  2. .vs/Binaries/DerivedDataCache/Intermediate/Saved/.sln 파일 없애고, uproject에서 VS sln파일 다시 생성 후, 솔루션 파일 빌드하기

<br/>

### ⚠️ 파일 내 오류
0. 솔루션 파일 빌드했을 때 Error 확인 후 구글링 (본인의 경우 LINK2005가 떴음)
1. .cpp, .h 파일 내 오타가 있어 수정했다.
2. Error LINK2005: 다중 정의(Multiple Definition) 에러 [➡️참고](https://stackoverflow.com/questions/15421254/already-defined-in-obj-no-double-inclusions)
3. LINK2005 에러는 **함수 정의가 중복될 때 발생**한다. 이는 주로 **헤더 파일(.h)에 함수 구현(정의)을 포함**하고, 이 헤더를 여러 소스 파일(.cpp)에서 `include`할 때 일어난다. 각 `.cpp` 파일이 독립적으로 함수를 정의하면서, **링크 단계에서 중복된 정의를 발견**해 에러를 내보낸다. 해결책은 **함수 선언은 `.h` 파일에, 구현은 `.cpp` 파일에 분리**하는 것이다.
4. 자세한 내용은 [➡️LINK2005](./LINK2005.md)
