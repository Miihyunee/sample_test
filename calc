/**
 * 한국환경보전원 출장 여비 계산 로직
 * 여비규정 [시행 2025.12.18.] 기준
 *
 * 브라우저: <script src="calc.js"></script> 로 로드
 * Node.js:  require('./calc.js') 로 로드
 */

// 기본 단가 (config.json 미로드 시 또는 Node.js 테스트 환경 폴백)
var DEFAULT_CONFIG = {
  effectiveDate: '2025.12.18.',
  outer: { ilbiPerDay: 25000, ilbiVehiclePerDay: 12500, mealPerDay: 25000,
           lodgingCap: { '서울': 100000, '광역시': 80000, '기타': 70000 } },
  inner: { ilbi4hPlus: 20000, ilbiUnder4h: 10000, vehicleDeduct: 10000 }
};

function calcTrip(data, cfg) {
  var c = cfg || (typeof TRIP_CONFIG !== 'undefined' ? TRIP_CONFIG : DEFAULT_CONFIG);
  var lodgingCap = c.outer.lodgingCap;
  var isInner = data.tripType.indexOf('내') === 0;

  // 차량 이용 일수: 미제공 시 전체 일수로 간주 (하위호환)
  var vDays = 0;
  if (data.vehicle !== 'public') {
    vDays = (data.vehicleDays !== undefined)
      ? Math.min(Math.max(0, data.vehicleDays), data.days)
      : data.days;
  }
  var pubDays = data.days - vDays;

  var ilbi = 0, meal = 0, lodging = 0, transport = data.transport;
  var ilbiPerDay = 0, ilbiVehiclePerDay = 0, mealDeduct = 0, mealFull = 0;

  if (isInner) {
    ilbiPerDay = (data.tripType === '내4') ? c.inner.ilbi4hPlus : c.inner.ilbiUnder4h;
    if (data.vehicle === 'company') ilbiVehiclePerDay = Math.max(0, ilbiPerDay - c.inner.vehicleDeduct);
    else if (data.vehicle === 'exclusive') ilbiVehiclePerDay = 0;
    else ilbiVehiclePerDay = ilbiPerDay;
    ilbi = ilbiPerDay * pubDays + ilbiVehiclePerDay * vDays;
  } else {
    ilbiPerDay = c.outer.ilbiPerDay;
    if (data.vehicle === 'company') ilbiVehiclePerDay = c.outer.ilbiVehiclePerDay;
    else if (data.vehicle === 'exclusive') ilbiVehiclePerDay = 0;
    else ilbiVehiclePerDay = c.outer.ilbiPerDay;
    ilbi = ilbiPerDay * pubDays + ilbiVehiclePerDay * vDays;

    mealFull = c.outer.mealPerDay * data.days;
    mealDeduct = Math.floor(data.freeMeals * (c.outer.mealPerDay / 3));
    meal = Math.floor(Math.max(0, mealFull - mealDeduct) / 10) * 10;

    if (data.nights > 0) {
      if (data.grade === '1') {
        lodging = data.lodgingPerNight * data.nights;
      } else {
        var cap = lodgingCap[data.region];
        lodging = Math.min(data.lodgingPerNight, cap) * data.nights;
      }
    }
  }

  var total = ilbi + meal + lodging + transport;
  return { ilbi: ilbi, meal: meal, lodging: lodging, transport: transport, total: total,
           ilbiPerDay: ilbiPerDay, ilbiVehiclePerDay: ilbiVehiclePerDay,
           vDays: vDays, pubDays: pubDays,
           mealDeduct: mealDeduct, mealFull: mealFull, isInner: isInner };
}

/**
 * 개인예금/법인예금 분리 계산
 * @param {object} result - calcTrip() 결과
 * @param {string} payment - '개인카드' | '법인카드'
 * @returns {{ personal: number, corp: number }}
 */
function calcDeposit(result, payment) {
  var personal = result.ilbi + result.meal;
  var corp = 0;
  if (payment === '개인카드') {
    personal += result.lodging + result.transport;
  } else {
    corp = result.lodging + result.transport;
  }
  return { personal: personal, corp: corp };
}

/**
 * 근무지 내/외 판정 (본원: 서울특별시 성동구)
 * 제2조 7항: 같은 시(특별시, 광역시 및 특별자치시를 포함)·군 내 출장 = 근무지 내
 * @param {string} regionName - 지역명 (예: '서울특별시', '인천광역시', '경기도')
 * @returns {boolean} true = 근무지 내
 */
function isInnerTrip(regionName) {
  return regionName.indexOf('서울') === 0;
}

if (typeof module !== 'undefined') module.exports = { calcTrip: calcTrip, calcDeposit: calcDeposit, isInnerTrip: isInnerTrip };
