## JSON PATH

- JSON 데이터를 조회하기 위한 언어

```json
{
  "car": {
    "color": "blue",
    "price": "$20,000"
  },
  "bus": {
    "color": "white",
    "price": "$120,000"
  }
}
```

- 일반 조회: `car.color`
- 루트 조회: `$.car.color`
- 리스트 조회: `$[<index>]`
- 조건 (값이 40 초과인 요소): `$[?(@ > 40)]`
- 조건 (location이 "rear-right"인 휠 모델): `$.car.wheels[?(@.location=="rear-right")].model`
- 모든 요소의 필드: `$.*.color`
- 리스트 내 모든 요소의 필드: `$[*].model`
- 인덱스 0과 3 요소 선택: `$[0,3]`
- 슬라이스 (start~end): `$[0:3]`
- 슬라이스 스텝: `$[0:3:2]`
- 마지막 요소: `$[-1]` 또는 `$[-1:]`

- `kubectl get nodes -o=jsonpath='<JSON-PATH>'`: 특정 조건을 만족하는 요소만 선택
- `-o=custom-columns=<COLUMN_NAME>:<JSON-PATH>`: 특정 컬럼만 선택
- `--sort-by=<JSON-PATH>`: 특정 path를 정렬 기준으로 사용
