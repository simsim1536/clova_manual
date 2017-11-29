## CFR API란?

Clova Face Recognition API(CFR API)는 이미지 데이터를 입력받은 후 얼굴 인식 결과를 JSON 형태로 반환한다. CFR API는 이미지에 있는 얼굴을 인식하여 분석 정보를 제공하는 얼굴 감지 API와 닮은 연예인을 알려주는 유명인 얼굴 인식 API를 제공한다. CFR API는 HTTP 기반의 REST API이며, 사용자 인증(로그인)이 필요하지 않은 비로그인 Open API이다.

***
## 사전 준비사항

CFR API를 사용하려면 개발하려는 애플리케이션을 네이버 개발자 센터에 등록해야 한다. 이때, 사용할 API에 대한 권한을 설정해야 하며, API 사용 시 필요한 정보를 획득해야 한다. 애플리케이션 등록을 참고하여 다음과 같이 필요한 사항을 미리 준비한다.

---

## 애플리케이션 등록(API 이용신청)

애플리케이션의 기본 정보를 등록하면, 좌측 내 어플리케이션 메뉴의 서브 메뉴의 등록한 애플리케이션 이름으로 서브 메뉴가 만들어진다.


1. **애플리케이션 등록하기**를 통해 앱 등록 페이지로 이동한다.
2. **사용 API**에서 **얼굴인식**(CFR API)를 선택한다.
3. **비로그인 오픈 API서비스 환경**에 **WEB 설정**을 추가하고 웹 애플리케이션의 **웹 서비스 URL**을 입력한다.
4. **등록하기** 버튼을 클릭하면 **내 애플리케이션** 메뉴로 이동하며 방금 등록한 앱의 정보가 화면에 표시된다. 이 페이지에서 **Client ID**와 **Client Secret** 정보를 확인할 수 있다.

***

## CFR API 사용하기

CFR API는 REST API이며, 얼굴 인식을 수행할 이미지 데이터를 HTTP 통신으로 음성 합성 서버에 전달한다. 음성 합성 서버가 제공하는 REST API의 URI는 다음과 같다.

! [CFR1](./img/CFR1.png)

HTTP 요청으로 얼굴 인식을 요청할 때 **사전 준비사항**에서 발급받은 Client ID와 Client Secret 정보를 헤더에 포함시켜야 한다. 또한 요청을 multipart 형식으로 보내야 하며, 메시지의 이름은 image여야 한다. 다음은 유명인 얼굴 인식 API를 호출할 때 보내는 HTTP 요청 메시지의 예이다.

! 이미지 파일 이름 URL

위와 같은 HTTP 요청을 얼굴 인식 서버로 전달하면 얼굴 인식 서버는 JSON 형태의 분석 결과 데이터를 HTTP 응답 메시지로 반환합니다. 다음은 응답 예제이다.

! 이미지 파일 이름 URL

위와 같은 방식으로 얼굴 감지 API도 사용할 수 있다.

***
## 구현 예제

import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLConnection;

// 네이버 얼굴인식 API 예제
public class APIExamFace {

    public static void main(String[] args) {

        StringBuffer reqStr = new StringBuffer();
        String clientId = "YOUR_CLIENT_ID";//애플리케이션 클라이언트 아이디값";
        String clientSecret = "YOUR_CLIENT_SECRET";//애플리케이션 클라이언트 시크릿값";

        try {
            String paramName = "image"; // 파라미터명은 image로 지정
            String imgFile = "이미지 파일 경로 ";
            File uploadFile = new File(imgFile);
            String apiURL = "https://openapi.naver.com/v1/vision/celebrity"; // 유명인 얼굴 인식
            //String apiURL = "https://openapi.naver.com/v1/vision/face"; // 얼굴 감지
            URL url = new URL(apiURL);
            HttpURLConnection con = (HttpURLConnection)url.openConnection();
            con.setUseCaches(false);
            con.setDoOutput(true);
            con.setDoInput(true);
            // multipart request
            String boundary = "---" + System.currentTimeMillis() + "---";
            con.setRequestProperty("Content-Type", "multipart/form-data; boundary=" + boundary);
            con.setRequestProperty("X-Naver-Client-Id", clientId);
            con.setRequestProperty("X-Naver-Client-Secret", clientSecret);
            OutputStream outputStream = con.getOutputStream();
            PrintWriter writer = new PrintWriter(new OutputStreamWriter(outputStream, "UTF-8"), true);
            String LINE_FEED = "\r\n";
            // file 추가
            String fileName = uploadFile.getName();
            writer.append("--" + boundary).append(LINE_FEED);
            writer.append("Content-Disposition: form-data; name=\"" + paramName + "\"; filename=\"" + fileName + "\"").append(LINE_FEED);
            writer.append("Content-Type: "  + URLConnection.guessContentTypeFromName(fileName)).append(LINE_FEED);
            writer.append(LINE_FEED);
            writer.flush();
            FileInputStream inputStream = new FileInputStream(uploadFile);
            byte[] buffer = new byte[4096];
            int bytesRead = -1;
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                outputStream.write(buffer, 0, bytesRead);
            }
            outputStream.flush();
            inputStream.close();
            writer.append(LINE_FEED).flush();
            writer.append("--" + boundary + "--").append(LINE_FEED);
            writer.close();
            BufferedReader br = null;
            int responseCode = con.getResponseCode();
            if(responseCode==200) { // 정상 호출
                br = new BufferedReader(new InputStreamReader(con.getInputStream()));
            } else {  // 에러 발생
                System.out.println("error!!!!!!! responseCode= " + responseCode);
                br = new BufferedReader(new InputStreamReader(con.getInputStream()));
            }
            String inputLine;
            if(br != null) {
                StringBuffer response = new StringBuffer();
                while ((inputLine = br.readLine()) != null) {
                    response.append(inputLine);
                }
                br.close();
                System.out.println(response.toString());
            } else {
                System.out.println("error !!!");
            }
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
