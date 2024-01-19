---
layout: single
title: "AWS CloudFront"
categories: [AWS]
tag: [aws, saa, cloudfront, region]
---

<div class="notice">
    <h4> 💡 참고: </h4>
    <ul>
        <li>
            <a href="https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Introduction.html"> Amazon CloudFront란 무엇입니까? </a>
        </li>
        <li>
            <a href="https://bosungtea9416.tistory.com/entry/AWS-CloudFront"> [AWS] CloudFront 에 대하여 </a>
        </li>
        <li>
            AWS SAA-03 Practice Exams (Udemy)
        </li>
    </ul>
</div>

## regional edge cache(리전 엣지 캐시)
<span style='color:black;background-color: #fff5b1'>리전 엣지 캐시(REC; Regional Edge Cace)</span>는, 사용자가 접근할 수 있으며 글로벌 하게 배포된 CloudFront 위치(locations)를 말한다. Origin[^1]과 Edge Location[^2] 사이에 위치해 있다.

제공할 콘텐츠가 캐시가 유지될 정도로 인기 있지 않아도 **캐시는 더 오랫동안 남으며, Edge Location 보다 캐시 스토리지 용량이 크다.** 즉, 리전 엣지 캐시는 모든 유형의 콘텐츠 -- 특히 시간이 지나면서 점차 사용되지 않게 되는 콘텐츠에 유용하다. <span style='color:gray'>이러한 콘텐츠의 예로는 동영상, 사진 또는 아트워크와 같은 사용자 생성 콘텐츠, 제품 사진 및 동영상과 같은 전자 상거래 자산, 갑자기 사용자가 많아질 수 있는 뉴스 및 이벤트 관련 콘텐츠 등이 있다.</span>


[^1]: **Origin:** Origin server; 콘텐츠가 위치하고 있는 근원을 말한다. <span style='color:gray'>(ex. EC2 Instance, S3 Bucket, On-Premise...)</span>
[^2]: **Edge Location:** CloudFront 서비스가 콘텐츠를 캐싱하고, Client에게 제공하는 지점 혹은 캐시 서버를 말한다. POP(Point of Presence)라고도 한다.


### 리전 엣지 캐시 작동 방식
> [CloudFront에서 리전 에지 캐시를 사용하는 방식](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/HowCloudFrontWorks.html#CloudFrontRegionaledgecaches)

객체의 사용률이 떨어지면, 개별 Edge Location이 이러한 객체를 제거하고, 확보된 공간은 사용률이 높은 콘텐츠가 사용하도록 한다. 하지만, 위에서 언급했듯이, 리전 엣지 캐시는 개별 Edge Location보다 캐시 공간이 더 크다. 즉, 객체가 가장 가까운 리전 엣지 캐시 위치에 더 오래 캐시 상태로 유지된다.

따라서, 더 많은 콘텐츠를 최종 사용자로 부터 가까운 거리가 되도록 유지할 수 있다. CloudFront가 Origin으로 돌아가야 할 필요성이 줄어들고, 최종 사용자를 위한 전반적인 성능도 향상된다.



##### **리전 엣지 캐시를 통해 <u>배포되는</u> 유형의 콘텐츠:**
1. 제품 사진 등의 e-commerce assets
2. 사용자 제작 비디오(User-generated videos)
3. style sheets, JavaScript 파일 등의 정적 콘텐츠

##### **리전 엣지 캐시를 통해 <u>배포되지 않는</u> 유형의 콘텐츠:**
1. 요청 시 결정된 동적 콘텐츠(모든 헤더를 전달하도록 구성된 cache-behavior)<br>
    <span style='color:gray'>(*원문: Dynamic content, as determined at request time (cache-behavior configured to forward all headers))</span>
2. 프록시 메서드 PUT/POST/PATCH/OPTIONS/DELETE는 origin으로 바로 이동한다<br>
    <span style='color:gray'>(*원문: Proxy methods PUT/POST/PATCH/OPTIONS/DELETE go directly to the origin)</span><br>
    → 지역 엣지 캐시를 통해 프록시하지 않는다.
3. Origin이 S3 버킷이고, 요청의 최적 REC가 S3 버킷과 동일한 리전에 있는 경우, Edge Location이 REC를 건너 뛰고 S3 버킷으로 직접 이동한다.

## CloudFront 동작 순서
> [CloudFront에서 콘텐츠를 제공하는 방법](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/HowCloudFrontWorks.html)

![cloudfront-action-sequence]({{site.url}}/images/2024-01-18-aws-cloudfront/cloudfront-action-sequence.png){: width="60%" height="60%"}

1. **사용자가** 애플리케이션에 액세스 하고, 하나 이상의 객체에 대한 **요청을 보낸다**.
2. **DNS는** 사용자에게 적합한 CloudFront POPs(Edge Location)[^3]으로 라우팅한다.[^4]
3. **CloudFront는** 캐시에 요청된 객체가 있는지 확인한다.
    1. 객체가 캐시에 *있으면*, 이것을 사용자에게 반환한다.
    2. 객체가 캐시에 *없으면*, 일반적으로 가장 가까운 Region Edge Cache(REC)에 캐시가 있는지 요청한다.
    3. REC에도 없으면 CloudFront는 Origin server<span style='color:gray'>(ex. S3 버킷 또는 HTTP 서버)</span>로 요청을 전달한다.<br>
4. **origin server는** `Origin > REC > Edge Location > CloudFront가 사용자에게 전달` 수순을 밟는다. (이 때, 캐시도 추가된다)
    1. REC에 캐시가 있다면, REC는 콘텐츠를 요청한 Edge Location으로 반환한다.
    2. Origin으로부터 REC에 콘텐츠의 첫 번째 byte가 도착하는 즉시, Edge Location은 이를 사용자에게 반환한다.
    3. Edge Location은 나중을 위해 **이 콘텐츠 캐시를 저장**한다.

이후 콘텐츠는 TTL동안 Edge Location에 캐싱되어 낮은 지연 시간(low-latency)으로 콘텐츠를 요청할 수 있다.

---

[^3]: **CloudFront POPs:** Point of Presence; 상호 접속 위치. Edge Location이라고도 함.
[^4]: 사용자에게 최적의 서비스를 제공할 수 있는 POP; 일반적으로 지연 시간과 관련해 가장 가까운 POP를 말하며, 요청을 Edge Location으로 라우팅한다.