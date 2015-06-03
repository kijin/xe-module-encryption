
XE 암호화 모듈
==============

XE 모듈이나 애드온 개발자를 위한 대칭키(AES) 및 비대칭키(RSA) 암호화 루틴을 제공하는 모듈입니다.
암호화 기능이 필요한 모듈이나 애드온에서 직접 호출하여야 하며,
관련 루틴을 호출하지 않는 다른 모듈의 데이터를 자동으로 암호화해 주지는 않습니다.

모듈 설정 페이지에서 키를 생성한 후에 사용해야 합니다.
키는 서버의 난수 생성기(`/dev/urandom`)를 사용하여 자동으로 생성되며,
DB 또는 별도의 파일에 저장하도록 설정할 수 있습니다.

대칭키 암호화를 사용하려면 `mcrypt` 모듈이 필요하며,
비대칭키 암호화를 사용하려면 `openssl` 모듈이 설치되어 있어야 합니다.
PHP 5.2 이상이면 대부분의 기능이 정상 작동하지만,
최적의 실행 환경을 위해서는 PHP 5.4 이상을 권장합니다.

대칭키 암호화 (AES)
-------------------

암호화와 복호화에 동일한 비밀키(암호)를 사용합니다.
빠르고 강하고 효율적인 알고리듬으로, 대부분의 암호화 작업에 적합합니다.

128, 192, 256비트의 키 길이와 128, 192, 256비트의 HMAC 길이를 선택할 수 있습니다.
키 길이는 언제든지 변경할 수 있습니다.

### 암호화

	$plaintext = '내일 오후 3시에 아지트에서 만나자';
	$oEncryption = getModel('encryption');
    $ciphertext = $oEncryption->aesEncrypt($plaintext);

### 복호화

    $ciphertext = '암호화된 문자열';
    $oEncryption = getModel('encryption');
    $plaintext = $oEncryption->aesDecrypt($ciphertext);

비대칭키 암호화 (RSA)
---------------------

개인키로 암호화한 데이터는 공개키로 복호화되고, 공개키로 암호화한 데이터는 개인키로 복호화됩니다.
암호화에 사용한 키를 노출하지 않고도 서로 데이터를 주고받을 수 있다는 장점이 있지만,
대칭키 알고리듬보다 효율이 떨어집니다. 비트수가 높더라도 실제로는 256비트 대칭키 암호화 알고리듬보다 약합니다.

1024, 2048, 3072, 4096비트의 키 길이와 128, 192, 256비트의 HMAC 길이를 선택할 수 있습니다.
한 번 키를 생성한 후에는 키 길이를 변경할 수 없습니다.

### 개인키를 사용한 암호화

	$plaintext = '내일 오후 3시에 아지트에서 만나자';
	$oEncryption = getModel('encryption');
    $ciphertext = $oEncryption->rsaEncryptWithPrivateKey($plaintext);

### 개인키를 사용한 복호화

    $ciphertext = '암호화된 문자열';
    $oEncryption = getModel('encryption');
    $plaintext = $oEncryption->rsaDecryptWithPrivateKey($ciphertext);

### 공개키를 사용한 암호화

	$plaintext = '내일 오후 3시에 아지트에서 만나자';
	$oEncryption = getModel('encryption');
    $ciphertext = $oEncryption->rsaEncryptWithPublicKey($plaintext);

### 공개키를 사용한 복호화

    $ciphertext = '암호화된 문자열';
    $oEncryption = getModel('encryption');
    $plaintext = $oEncryption->rsaDecryptWithPublicKey($ciphertext);

암호문 포맷
-----------

### 버전 1.1

  - AES: `AE`로 시작하는 문자열
  - RSA 개인키: `RA`로 시작하는 문자열
  - RSA 공개키: `RB`로 시작하는 문자열

### 버전 1.0

  - AES: `AK`로 시작하는 문자열
  - RSA 개인키: `RP`로 시작하는 문자열
  - RSA 공개키: `RU`로 시작하는 문자열

상위 버전의 모듈은 하위 버전의 포맷을 지원합니다.
상위 버전에서 암호화된 데이터는 하위 버전에서 복호화하지 못할 수도 있습니다.

버전 1.1에서는 `mcrypt` 모듈의 잘못된 알고리듬 선택 및 널바이트 패딩 버그를 우회하는 코드를 적용하였으며,
평문을 불필요하게 압축하지 않도록 변경하였습니다.

기타
----

암호가 있는 개인키를 사용하려면 비대칭 암호화 키를 파일에 저장하도록 설정한 후,
별도로 생성한 키 조합을 해당 파일에 덮어씌우면 됩니다.
개인키를 이용한 암호화 및 복호화 작업시에 두 번째 인자로 개인키 암호가 들어갑니다.

암호화할 때는 자동으로 HMAC을 생성하여 데이터 위변조를 방지하므로,
암호화된 데이터를 쿠키에 넣거나 폼으로 주고받아도 안전합니다.

키를 변경하거나 삭제하면 기존의 키로 암호화된 데이터는 더이상 해독할 수 없게 되니 주의하시기 바랍니다.

이 모듈은 XE와 동일한 LGPL v2.1 라이선스로 배포됩니다.
