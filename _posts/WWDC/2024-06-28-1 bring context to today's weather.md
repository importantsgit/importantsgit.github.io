---
title: "WWDC24_Bring context to today's weather"
excerpt: "WWDC24 내 weatherKit에 관한 Post입니다."
layout: single
categories:
  - WWDC_24
author_profile: false
sidebar:
  nav: "counts"
permalink: /category/today/wwdc/weather

date: 2024-06-28 # 작성 날짜
last_modified_at: 2024-06-28 # 최종 수정 날짜
---

# weatherKit

Apple weather service를 기반으로 하여, 정확한 지역 날씨 데이터를 제공하는 API 입니다. <br><br><br>

## 요약

1. 더 풍부한 예보 세부 정보를 보여줍니다.

   - 강수량, 고도별 구름 덮개 등의 추가 정보 제공
   - 시간별 강수량 데이터와 낮과 밤을 구분한 일일 예보 추가
     <br><br>

2. 변화를 감지하거나, 그 동안의 History를 비교할 수 있습니다.

   - 날씨 변화 감지 기능 추가 (온도나 강수량 변화 시점)
   - 과거 평균과의 비교 기능 추가: 현재 날씨와 History 평균의 편차 비교
     <br><br>

3. 새로운 통계 API가 나왔습니다.

   - 1970년 이후의 장기 평균 기온과 강수량 데이터를 시간별, 일별, 월별로 제공
   - 일일 요약 데이터를 통해 특저 날짜의 최고/최저 기온과 강수량 제공
     <br><br>

4. 데이터 전송을 최적화했습니다.

   - FlatBuffers 지원: JSON 대비 최대 25%의 payload 크기 감소 및 90%의 파싱 시간 감소
     <br><br><br>

## 올해의 개선 사항

### Richer forecast details

강수량, 고도별 구름량 등의 세부정보를 통해 예측을 강화하고 있습니다.
<br><br>
weatherKit은 여러 가지 훌륭한 API로 구성되어 있습니다.

- availability: 특정 위치에 사용할 수 있는 데이터셋을 알려줍니다.
- attribution: 데이터 출처를 제공합니다.
- activeAlert: 현재 진행 중인 심각한 날씨에 대한 세부 정보를 제공합니다.

<br>
그리고 가장 중요한 ***Weather API***가 있습니다.<br>
이 API는 현재 날씨 상황, 분 단위 강수량, 10일간의 시간별 및 일일 예보, 그리고 날씨 경고 등 가장 필수적인 날씨 데이터를 제공합니다.<br>
그리고 올해는 현재, 시간별 및 일일 에보에 더 많은 세부 정보를 추가하여 Weatherkit을 개선하고 있습니다.<br>
현재 날씨 조건은 온도, 풍속, 자외선 지수와 같은 순간의 날씨 정보를 제공하고 있습니다. <br><br>
(***NEW***) 그리고, 고도별 구름 덮개와 같은 더 세부적인 정보를 추가할 수 있습니다.<br>
<img src="/assets/images/posts/WWDC/24_weather/weather_current.PNG" />

이러한 개선 사항은 시간별 예보에도 적용됩니다.
시간별(hourly) 구름 덮개 예보에 고도별 변화를 추가하거나, Apple Weather 앱처럼 시간별 강수량을 제공할 수 있습니다.
<img src="/assets/images/posts/WWDC/24_weather/weather_hourly.PNG" /><br><br>
또한, Daily 예보 쿼리가 가장 큰 개선을 받았습니다.
이제 높은 습도와 낮은 습도와 같은 더 세분화된 일일 예보를 앱에 쉽게 추가할 수 있습니다.<br>
또는 낮과 밤의 날씨를 분리하여 일일 예보를 분할할 수도 있습니다.
<img src="/assets/images/posts/WWDC/24_weather/weather_daily.PNG" />

**강수량 데이터를 가져오는 예시**

```swift
extension CLLocation {
    static var newYork: CLLocation { CLLocation(latitude: 50.710, longitude: -74.023) }
}

let hourlyPercipitation = try await WeatherService()
    .weather(for: .newYork, including: .hourly)
    .map(\.precipitationAmount)
```

1. 먼저 데이터를 가져오려는 위치를 전달해야 합니다.
2. 그런 다음 데이터의 세분화 수준을 지정합니다.
3. 그 후, 시간별 예보의 precipitationAmount 속성에 간단히 액세스 합니다.

이러한 속성은 REST API를 통해 사용할 수 있습니다.<br>
waetherKit API는 weather.apple.com에 호스팅되어 있으므로 RESTful URL 경로로 해당 주소를 전달합니다.<br>
<img src="/assets/images/posts/WWDC/24_weather/restful api.PNG" />
그리고 Apple Weather Service에 액세스하려면 인증 베어러를 생성해야 한다는 점을 기억하세요.<br>
해당 URL에 GET 요청을 보내면 forecastHourly 키 아래에 호스팅된 객체가 반환됩니다.<br>

### Highlight weather changes

예측을 통해 도출된 날씨 변화를 상황에 맞게 강조 표시하는 기능을 도입합니다.<br>

계획을 세우기 위해 중요한 날시 변화 패턴을 적극적으로 찾고 있습니다.<br>
이것은 Changes와 Historical Comparisons 이 새로운 Weather로 체크가 가능합니다.<br>

<img src="/assets/images/posts/WWDC/24_weather/newAPI.PNG" />
<br><br><br>
#### Changes

변화 쿼리는 온도와 강수량의 다가오는 변화를 제공하며, 발생 시점과 같은 중요한 다가오는 변동을 알려줍니다.
이 쿼리를 사용하여 사용자에게 다가오는 온도 변화를 강조할 수 있습니다.

```swift
let changes = try await WeatherService()
    .weather(for: .newYork, including: .change)

let lowtemperatureChanges = changes?
    .filter(\.date.isTomorrow )
    .map(\.lowTemperature)

if let lowTemperatureChanges,
   lowTemperatureChanges.contains(.decrease) {
     // 만약 낮은 온도 값이 더 떨어진다면
   }
```

이러한 속성은 REST로도 구현이 가능합니다.
<img src="/assets/images/posts/WWDC/24_weather/change.PNG" />
<br><br><br>

#### Historical comparisons

지난 50년 이상 수집된 데이터를 기반으로 Historical Comparisons는 오늘의 날씨를 역사적 평균과 비교할 수 있게 하며, 평균으로부터의 편차 중요도에 따라 결과를 정렬합니다.
목록은 평균으로부터의 편차 중요도에 따라 정렬됩니다.

```swift
let mostSignificant = try await WeatherService()
    .weather(for: .newYork, including: .historicalComparisons)?

switch mostSignificant {
    case .highTemperature(let trend), .lowTemperature(let trend):

    case .some, .none:
        break

}
```

이러한 속성은 REST로도 구현이 가능합니다.
<img src="/assets/images/posts/WWDC/24_weather/historical.PNG" />
<br><br><br>

### static API

특정 위치의 장기적인 날씨 패턴을 구하고 싶을 겁니다.<br>

이를 쉽게 하기 위해 넓고 좁은 역사적 날씨 통계를 얻을 수 있도록 설계된 새로운 Statistics API를 추가하고 있습니다.<br>

Statistics API에는 Historical Averages와 Daily Summaries의 두 가지 새로운 쿼리가 있습니다.<br> Historical Averages를 사용하면 1970년 1월 1일 이후 기록된 데이터를 기반으로 기온과 강수량의 장기 평균을 가져올 수 있습니다.<br>

```swift
let averagePrecipitation = try await WeatherService()
    .monthStatistics(
        for: .newYork,
        startMonth: 1,
        endMonth: 12,
        including: .precipitation
    )

let averagePrecipitationAmountsPerMonth = Dictionary(
    grouping: averagePrecipitation,
    by: \.month
)
```

마찬가지로, REST로도 가능합니다.<br>
<img src="/assets/images/posts/WWDC/24_weather/staticAPI.PNG" />
<br><br>
또는, 모든 데이터가 아닌 해당 기간동안의 요약을 알고 싶을 때도 있을 겁니다.

```swift
let pastThirtyDaysSummary = try await WeatherService()
    .dailySummary(
        for: .newYork,
        forDaysIn: DateInterval(startL .thirtyDaysAgo, end: .now)
    )
    .first

if let pastThirtyDaysSummary {}
```

<img src="/assets/images/posts/WWDC/24_weather/weatherSummary.PNG" />

### Faster data transfer

마지막으로 데이터 전송을 가속화하는 바이너리 형식에 대한 지원을 추가하고 있습니다.
