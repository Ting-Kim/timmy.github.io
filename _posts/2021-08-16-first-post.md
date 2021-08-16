---
layout: post
title: 첫번째 포스트
categories: 김애용
tags: [김애용]
---

엄청 헤맸는데 이제서야 대충 정리가 된듯.

참고한 사이트 : [Travis CI로 Jekyll & Github Pages 배포 자동화 하는 방법](https://deeplify.dev/tools/ci/github-pages-travis-ci)

결론은 github와 jekyll을 이용해 블로그를 개설해 사용할 때 보안적인 이슈로 인해서 github에서 특정 plugin만 build 허용을 한다(whitelisted plugins). 따라서 third party plugin을 사용하기 위해서는 jekyll build 완료한 결과물을 git repo에 업로드하여야 한다.

이 과정을 편하게 하기 위해서 Travis CI, Github action 을 사용하면 되고 어짜피 둘 다 안 써본 상황에서 github action을 예전부터 써보고 싶기도 했고, 친절하게 jekyll theme 제작자가 action의 workflows 파일까지 제공을 해줘서 github action으로 진행하였다.

```
내용이 Jekyll 빌드 전 상태인 경우, 자동 빌드 및 배포 (커스텀 플러그인 X)
내용이 Jekyll 빌드 완료 후 결과물인 경우, 그대로 웹 페이지 노출 (커스텀 플러그인 O)
```

즉, 내가 jekyll 빌드 전 상태의 결과물을 master branch에 올리면 github action으로 jekyll 빌드가 이뤄지고 gh-pages라는 branch로 빌드 결과물이 올라간다. 이 branch를 호스팅함으로써 third party plugin을 사용할 수 있다.

솔직히 안 써도 되고, 쓸 일도 없을 것 같긴한데 Jekyll Spaceship이라는 plugin이 MathJax, Diagram, 영상 등을 편하게 제공해주는 듯 하기에 해보기로 함.

전반적인 스텝은 위 사이트에 있는 다른 글 [Github Action으로 Jekyll & Github Pages 배포 자동화 하는 방법](https://deeplify.dev/tools/git/github-pages-github-action)을 참고하였고 Jekyll YAT theme을 사용한다면 해당 repo에 이미 올라와있는 .github/workflows/build-jekyll.yml을 사용하면 된다.

gh-pages branch로 빌드 결과물이 생성되므로 repo-settings-pages-Github Pages-Source의 branch를 변경해줘야 되고, 위 workflows 파일의 jekyll_baseurl을 '' 으로 수정해줘야 된다. 이걸 놓쳐서 몇시간 날림.
