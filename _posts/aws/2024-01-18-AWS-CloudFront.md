---
layout: single
title: "AWS CloudFront"
categories: [AWS]
tag: [aws, saa, cloudfront, region]
---

<div class="notice">
    <h4> 💡 참고: </h4>
    <ul>
        <li> <a href="https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Introduction.html"> Amazon CloudFront란 무엇입니까? </a> </li>
        <li> <a href="https://bosungtea9416.tistory.com/entry/AWS-CloudFront"> [AWS] CloudFront 에 대하여 </a> </li>
    </ul>
</div>

## regional edge cache(리전 엣지 캐시)
<span style='color:black;background-color: #fff5b1'>리전 엣지 캐시(REC; Regional Edge Cace)</span>는, 사용자가 접근할 수 있으며 글로벌 하게 배포된 CloudFront 위치를 말한다. Origin[^1]과 Edge Location[^2] 사이에 위치해 있다.

콘텐츠가 캐시가 유지될 정도로 인기 있지 않아도 **캐시는 더 오랫동안 남으며, Edge Location 보다 캐시 스토리지 용량이 크다.**


[^1]: **Origin:** Origin server; 콘텐츠가 위치하고 있는 근원을 말한다. <span style='color:gray'>(ex. EC2 Instance, S3 Bucket, On-Premise...)</span>
[^2]: **Edge Location:** CloudFront 서비스가 콘텐츠를 캐싱하고, Client에게 제공하는 지점 혹은 캐시 서버를 말한다.


##### **리전 엣지 캐시를 통해 <u>배포되는</u> 유형의 콘텐츠:**
1. 제품 사진 등의 e-commerce assets
2. 사용자 제작 비디오(User-generated videos)
3. style sheets, JavaScript 파일 등의 정적 콘텐츠

##### **리전 엣지 캐시를 통해 <u>배포되지 않는</u> 유형의 콘텐츠:**
1. 요청 시 결정된 동적 콘텐츠(모든 헤더를 전달하도록 구성된 cache-behavior)<br>
    <span style='color:gray'>(*원문: Dynamic content, as determined at request time (cache-behavior configured to forward all headers))<span>
2. 프록시 메서드 PUT/POST/PATCH/OPTIONS/DELETE는 origin으로 직통한다<br>
    <span style='color:gray'>(*원문: Proxy methods PUT/POST/PATCH/OPTIONS/DELETE go directly to the origin)<span>

## CloudFront 동작 순서
![cloudfront-action-sequence]({{site.url}}/images/2024-01-18-aws-cloudfront/cloudfront-action-sequence.png){: width="60%" height="60%"}

1. 사용자가 애플리케이션에 요청을 보낸다.
2. DNS는 사용자에게 적합한 Edge Location으로 라우팅 한다.
3. Edge Location에서 캐시를 확인한다.
    1. 캐시가 있으면 이것을 사용자에게 반환한다.
    2. 없으면 가장 가까운 Region Edge Cache(REC)에 캐시가 있는지 요청한다.
        1. REC에도 없으면 CloudFront는 Origin server로 요청을 전달한다.
4. **origin server는** `Origin > REC > Edge Location > CloudFront가 사용자에게 전달` 수순을 밟는다. (이 때, 캐시도 추가된다)
    1. REC에 캐시가 있다면, REC는 콘텐츠를 요청한 Edge Location으로 반환한다.
    2. REC로부터 콘텐츠의 첫 번째 바이트가 도착하는 즉시, Edge Location은 이를 사용자에게 반환한다.
    3. Edge Location은 나중을 위해 **이 콘텐츠 캐시를 저장**한다.

이후 콘텐츠는 TTL동안 Edge Location에 캐싱되어 낮은 지연시간(low-latency)으로 콘텐츠를 요청할 수 있다.

---