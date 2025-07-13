# WiFi Text Scanner - Product Requirements Document (PRD)

## 1. 제품 개요

### 1.1 제품명
WiFi Text Scanner

### 1.2 제품 설명
카페, 공공장소 등에서 벽에 붙은 Wi-Fi 정보(SSID/비밀번호)를 수동으로 입력하지 않고, 모바일 앱으로 QR 코드 또는 텍스트를 스캔하여 자동으로 Wi-Fi에 연결할 수 있도록 지원하는 iOS/Android 전용 모바일 애플리케이션

### 1.3 목표 사용자
- 카페, 공공장소 등에서 Wi-Fi를 자주 사용하는 사용자
- 복잡한 Wi-Fi 비밀번호 입력에 불편함을 느끼는 사용자
- 빠르고 편리한 Wi-Fi 연결을 원하는 모든 스마트폰 사용자

### 1.4 핵심 가치
- **편의성**: 수동 입력 없이 카메라 스캔만으로 Wi-Fi 연결
- **정확성**: OCR 기술을 활용한 정확한 텍스트 인식
- **속도**: 빠른 스캔과 즉시 연결
- **모바일 최적화**: iOS/Android 네이티브 기능 활용

## 2. 기술 스택

### 2.1 개발 환경
- **플랫폼**: Replit (빠른 프로토타이핑)
- **프레임워크**: React Native with Expo
- **타겟 플랫폼**: iOS, Android (모바일 전용)
- **언어**: TypeScript
- **스타일링**: StyleSheet (React Native)

### 2.2 주요 라이브러리
- **카메라/스캔**: expo-camera, expo-barcode-scanner
- **OCR**: react-native-ml-kit 또는 expo-ml-kit-ocr
- **Wi-Fi 연결**: react-native-wifi-reborn (iOS/Android)
- **상태 관리**: React Hooks (useState, useEffect)

## 3. 핵심 기능

### 3.1 QR 코드 스캔
- Wi-Fi 정보가 담긴 QR 코드 자동 인식
- 표준 Wi-Fi QR 코드 형식 지원 (WIFI:T:WPA;S:SSID;P:password;;)
- 실시간 카메라 프리뷰 제공

### 3.2 텍스트 스캔 (OCR)
- 카메라로 Wi-Fi 정보 텍스트 촬영
- OCR을 통한 텍스트 추출
- SSID와 비밀번호 자동 구분 및 인식
- 다양한 텍스트 형식 지원:
  - "Wi-Fi: GuestNetwork / Password: 12345678"
  - "SSID: CafeWiFi\nPW: coffee2024"
  - 기타 일반적인 표기 형식

### 3.3 추출 정보 표시
- 스캔 완료 후 추출된 정보 화면 표시:
  - SSID (네트워크 이름)
  - 비밀번호
  - 보안 방식 (WPA/WPA2/WEP 등)
- 정보 수정 기능 제공 (OCR 오류 대비)

### 3.4 Wi-Fi 연결
- 추출된 SSID가 현재 감지되는 네트워크 목록에 있는지 확인
- 발견 시 연결 확인 팝업 표시:
  - "'{SSID}' 네트워크에 연결하시겠습니까?"
  - [연결] [취소] 버튼
- 연결 버튼 클릭 시 자동 Wi-Fi 연결 시도
- 연결 결과 표시 (성공/실패)

## 4. 사용자 인터페이스

### 4.1 메인 화면
- 카메라 프리뷰 영역
- 스캔 모드 전환 버튼 (QR/텍스트)
- 플래시 토글 버튼
- 갤러리에서 이미지 선택 옵션

### 4.2 결과 화면
- 추출된 Wi-Fi 정보 표시
- 정보 수정 가능한 입력 필드
- 연결 버튼
- 다시 스캔 버튼

### 4.3 설정 화면
- OCR 언어 설정
- 자동 연결 옵션
- 연결 기록 관리

## 5. 플랫폼별 제약사항

### 5.1 iOS
- **Wi-Fi 연결 제한**: iOS 11+ 에서는 NEHotspotConfiguration API 사용
- **권한 요구사항**:
  - 카메라 접근 권한
  - 위치 서비스 권한 (Wi-Fi 스캔 시 필요)
- **제약사항**:
  - 앱이 직접 Wi-Fi를 연결할 수 없고, 시스템 설정으로 이동
  - Wi-Fi 목록을 직접 스캔할 수 없음
  - 연결 성공 여부를 직접 확인하기 어려움

### 5.2 Android
- **권한 요구사항**:
  - 카메라 권한
  - 위치 권한 (Wi-Fi 스캔/연결 시 필수)
  - Wi-Fi 상태 변경 권한
- **제약사항**:
  - Android 10(API 29) 이상에서는 WifiNetworkSuggestion API 사용
  - 사용자가 제안된 네트워크 연결을 승인해야 함
  - Android 11+ 에서 더 엄격한 권한 정책

## 6. 기술적 구현 방안

### 6.1 QR 코드 처리
```javascript
// QR 코드 데이터 파싱 예시
const parseWifiQR = (data) => {
  // WIFI:T:WPA;S:MyNetwork;P:MyPassword;;
  const regex = /WIFI:T:([^;]*);S:([^;]*);P:([^;]*);/;
  const match = data.match(regex);
  if (match) {
    return {
      security: match[1],
      ssid: match[2],
      password: match[3]
    };
  }
};
```

### 6.2 OCR 텍스트 처리
```javascript
// OCR 결과에서 Wi-Fi 정보 추출
const extractWifiInfo = (ocrText) => {
  // 다양한 패턴 매칭
  const patterns = [
    /(?:SSID|Wi-?Fi|Network)[\s:]+([^\s\n]+)[\s\n]+(?:Password|PW|Pass)[\s:]+([^\s\n]+)/i,
    /([^\s\n]+)[\s\n]+(?:Password|PW|Pass)[\s:]+([^\s\n]+)/i
  ];
  // 패턴 매칭 로직
};
```

### 6.3 플랫폼별 Wi-Fi 연결
```javascript
// 플랫폼별 연결 처리
const connectToWifi = async (ssid, password) => {
  if (Platform.OS === 'ios') {
    // iOS: 설정 앱으로 이동
    Alert.alert(
      'Wi-Fi 연결',
      'iOS에서는 설정 앱에서 직접 연결해야 합니다.',
      [{ text: '설정 열기', onPress: () => Linking.openSettings() }]
    );
  } else if (Platform.OS === 'android') {
    // Android: WifiManager 사용
    await WifiManager.connectToProtectedSSID(ssid, password, false);
  }
};
```

## 7. MVP 범위

### 7.1 Phase 1 (MVP)
- [x] Expo 프로젝트 설정
- [ ] 카메라 권한 요청 및 프리뷰
- [ ] QR 코드 스캔 기능
- [ ] 스캔 결과 표시 화면
- [ ] 기본 UI/UX 구현

### 7.2 Phase 2
- [ ] OCR 텍스트 스캔 기능
- [ ] Wi-Fi 정보 추출 알고리즘
- [ ] Android Wi-Fi 연결 기능
- [ ] 연결 상태 표시

### 7.3 Phase 3
- [ ] iOS Wi-Fi 연결 지원
- [ ] 갤러리 이미지 스캔
- [ ] 연결 기록 저장
- [ ] 다국어 지원

## 8. 성공 지표

### 8.1 기술적 지표
- QR 코드 인식 성공률 > 95%
- OCR 텍스트 인식 정확도 > 90%
- 스캔-연결 완료 시간 < 10초

### 8.2 사용자 경험 지표
- 첫 사용 시 학습 시간 < 1분
- 수동 입력 대비 시간 절약 > 70%
- 사용자 만족도 > 4.0/5.0

## 9. 향후 계획

### 9.1 추가 기능
- 클라우드 동기화
- Wi-Fi 속도 테스트
- 안전한 네트워크 검증
- 공유 기능

### 9.2 비즈니스 모델
- 무료 버전: 기본 스캔 기능
- 프리미엄 버전: 무제한 기록, 클라우드 동기화
- 기업용: 대량 QR 코드 생성, 관리 도구

