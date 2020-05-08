2020-05-08
===


### json
<pre>
{
  "RETDATA": {
    "NCARDX": "7515-8820-2719-7551",
    "MKNAME": "한기완",
    "EVTCPCNT": 0,
    "POGYON": "0",
    "EVTCPURL": "http://devweb02a1.oilbank.co.kr:9080/m2012/login.do?RETURN_URL=%2Ffront%2FboardList.do%3Fpage_num%3D582%26pre_page_num%3D580%26bcode%3DMYPAGEEVENTCP"
  },
  "RETCODE": "00",
  "APPSESSIDRS": "Y",
  "RETSUMMARY": {
    "MONTH2": {
      "COUNT": "2",
      "MONEY": "260,000",
      "POINT": "4,535",
      "MONTH": "01"
    },
    "MONTH3": {
      "COUNT": "1",
      "MONEY": "32,000",
      "POINT": "80",
      "MONTH": "12"
    },
    "MONTH1": {
      "COUNT": "0",
      "MONEY": "0",
      "POINT": "0",
      "MONTH": "02"
    }
  },
  "RETCUSTNO": "P202001000008",
  "RETLIST": [
    {
      "ASALET": "200,000",
      "QOILAMT": "285",
      "HGURAE": "20.01.21",
      "POINT": "4,275",
      "POINTTYPE": "(유)스카이오일",
      "NO": 1
    },
    {
      "ASALET": "60,000",
      "QOILAMT": "52",
      "HGURAE": "20.01.20",
      "POINT": "260",
      "POINTTYPE": "(유)스카이오일",
      "NO": 2
    }
  ],
  "RETLOSTDATA": "0",
  "RETLIST2": [],
  "RETMSG": "SUCCESS!!!"
}
</pre>

<br>
대응 되는 dataclass
```
data class OilHisPointLucky004(
    var RETDATA: RETDATA004? = null,
    var RETCODE: String? = null,
    var APPSESSIDRS: String? = null,
    var RETSUMMARY: RETSUMMARY? = null,
    var RETCUSTNO: String? = null,
    var RETLIST: ArrayList<RETLIST004> = ArrayList<RETLIST004>(),
    var RETLOSTDATA: String? = null,
    var RETMSG: String? = null
)

data class RETLIST004(
    var ASALET: String? = null,
    var QOILAMT: String? = null,
    var HGURAE: String? = null,
    var POINT: String? = null,
    var POINTTYPE: String? = null,
    var NO: String? = null
)

data class RETSUMMARY(
    var MONTH2: MONTH2? = null,
    var MONTH3: MONTH3? = null,
    var MONTH1: MONTH1? = null,
    var MONTH6: MONTH6? = null,
    var MONTH4: MONTH4? = null,
    var MONTH5: MONTH5? = null
)

data class RETDATA004(
    var NCARDX: String? = null,
    var MKNAME: String? = null,
    var EVTCPCNT: String? = null,
    var POGYON: String? = null,
    var EVTCPURL: String? = null
)

data class MONTH1(
    var COUNT: String? = null,
    var MONEY: String? = null,
    var POINT: String? = null,
    var MONTH: String? = null
)

data class MONTH2(
    var COUNT: String? = null,
    var MONEY: String? = null,
    var POINT: String? = null,
    var MONTH: String? = null
)

data class MONTH3(
    var COUNT: String? = null,
    var MONEY: String? = null,
    var POINT: String? = null,
    var MONTH: String? = null

)


```
<br>

대응되는 retrofit
<pre>
<code>
 @FormUrlEncoded
    @POST(IF004_OILHIS_POINT_LUCKY)
    fun oilHisPointLucky(
        @Field("CUSTNO") CUSTNO: String? = null,
        @Field("APPSESSID") APPSESSID: String? = null,
        @Field("MGN") MGN: String? = "R",
        @Field("count") count: String? = null,
        @Field("NAPPVER") NAPPVER: String? = null,
        @Field("NOSTYPE") NOSTYPE: String? = null,
        @Field("NOSVER") NOSVER: String? = null
    ): Call<OilHisPointLucky004>
</code>
</pre>