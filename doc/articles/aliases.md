---
layout: default
title: "별칭(Aliases)"
permalink: /doc/articles/aliases.md/
---

# 별칭(Aliases)

## 별칭을 왜 쓰는가?

VCS 저장소(Version Control System repository)를 사용하는 경우, `2.0` 또는 `2.0.x`와 같은 버전에
대해 비슷한 브랜치들을 보유하게 됩니다. `master` 브랜치를 위해서 `dev-master` 버전을 보유할
수 있고, `bugfix` 브랜치를 위해서, `dev-bugfix` 버전을 보유할 수도 있습니다. 

만약 `master` 브랜치가 `1.0` ― `1.0.1`, `1.0.2`, `1.0.3`, 등 ― 개발 과정의 배포 과정에
태그 되도록 이용된다면, 이에 의존하는 어떤 패키지들은 아마도 `1.0.*` 버전을 요구하게 될 것입니다.

만약 최신 `dev-master`을 요구하는 어떤 패키지가 있다면, 다른 패키지들이 `1.0.*`에 의존하기 때문에
dev 버전이 충돌을 일으키는 문제가 발생합니다. 왜냐하면 `dev-master`는 `1.0.*` 조건에는 맞지 않기
때문입니다.

이 때 별칭을 입력합니다.

## 브랜치 별칭

여러분의 VCS 저장소에 `dev-master` 브랜치가 있다고 해 봅시다. 최신의 master의 개발 버전을 원하는 경우는 상당히 흔한 경우입니다. 이를 위해서 컴포저는 당신이 `dev-master`를 `1.0.x-dev` 버전으로 별칭을 지정할 수 있도록 합니다. 이렇게 하기 위해서는 `composer.json` 중 `extra` 아래 `branch-alias` 필드를 다음과 같이 지정하면 됩니다.

{% highlight json %}
{
    "extra": {
        "branch-alias": {
            "dev-master": "1.0.x-dev"
        }
    }
}
{% endhighlight %}

만약 호환되지 않는(non-comparible) 버전(dev-develop 같은)을 별칭하는 경우 브랜치 이름에 `dev-` 접두사를 붙여야 합니다. 또한 호환되는 버전(즉, 숫자로 시작하고, `.x-dev`로 끝나는)을 별칭할 수 있는데 좀 더 명확한 버전을 지정해야만 합니다. 예를 들면, 1.x-dev는 1.2.x-dev 로 별칭되어야 한다.

별칭은 호환되는 개발 버전이어야 하고, `branch-alias`(브랜치 별칭)는 이 별칭이 참고하는 브랜치에 존재해야 합니다. `dev-master`에 대해서, 당신은 이 브랜치 내용을 `master` 브랜치에 커밋 해야할 필요가 있습니다. 

결과적으로, 이제 어떤 패키지든 `1.0.*`를 요구하면 자연스럽게 `dev-master`를 설치하게 됩니다.

브랜치 별칭을 사용하기 위해서, 패키지 별칭을 붙인 저장소를 갖지고 있어야 합니다. 만약 유지 보수를 하지 않는 서드 파티 패키지의 한 포크(fork)를 별칭하고 싶은 경우, 다음에 설명하는 인라인 별칭(inline aliases)을 사용하면 됩니다.

## 인라인 별칭 요구하기

브랜치 별칭은 주요 개발 과정에서 훌륭하게 작동합니다. 하지만 별칭을 사용하기 위해서는 소스 저장소에 대한 제어를 할 수 있어야 하고, 버전 제어에 대한 변화를 커밋 할 수 있어야 합니다. 

만약 로컬 프로젝트가 의존하는 몇몇 라이브러리에 대해 단순히 버그 수정을 원하는 경우, 이는 정말 달갑지 않습니다.

이런 이유 때문에, 패키지 내부의 `require` 와 `require-dev` 필드에 별칭 지정할 수 있습니다. `monolog/monolog` 패키지에서 버그를 발견했다고 가정해봅시다. 여러분은 깃허브에서 [Monolog](https://github.com/Seldaek/monolog) 를 복제하고, `bugfix` 라는 이름의 브랜치에 버그를 수정했습니다. 이제 여러분은 버그를 수정한 monolog 를 여러분의 로컬 프로젝트에 설치하고 싶습니다.

여러분은 `monolog/monolog` 버전 `1.*`가 필요한 `symfony/monolog-bundle`을 이용하고 있고, 그래서 여러분이 생성한 `dev-bugfix`가 해당 조건(`1.*`)에 맞길 바라고 있습니다.

여러분의 프로젝트 최상의 폴더의 `composer.json` 에 다음을 추가하기만 하면 됩니다.

{% highlight json %}
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/you/monolog"
        }
    ],
    "require": {
        "symfony/monolog-bundle": "2.0",
        "monolog/monolog": "dev-bugfix as 1.0.x-dev"
    }
}
{% endhighlight %}

이렇게 하면, 여러분의 깃허브에서 `monolog/monolog`의 `dev-bugfix` 버전을 가지고 와서 `1.0.x-dev`라고 별칭할 것입니다. 

> **참고:** 만약 한 패키지에서 인라인 별칭이 요구되면, 그 별칭(코드 중 `as`의 오른쪽 부분)은 버전 조건으로 사용될 것입니다. 
> `as` 왼쪽 부분은 무시됩니다. 결과적으로, 만약 A가 B를 요구했고, B가 `monolog/monolog`의
> `dev-bugfix as 1.0.x-dev` 버전을 요구했다면, A를 설치하는 것은 B가 `1.0.x-dev`
> 를 요구하게 하고, `1.0.x-dev`는 실제 `1.0`에 존재할 수도 있습니다.
> 만약 존재하지 않는다면, A의 `composer.json`에서 다시-인라인-별칭(re-inline-aliased)지정을 합니다.


> **참고:** 인라인 별칭은 패키지를 배포하는 경우, 특히 피해야 합니다. 만약 당신이 버그를 찾는다면, 다음 커밋에서(upstream) 에서 수정한것이 반영될 수 있도록 머지하고 이 머지된 커밋을 가져오도록 하십시오. 
> 이렇게 하는 것이 당신 패키지 이용자들이 문제들을 겪지 않도록 하는 것입니다. 
