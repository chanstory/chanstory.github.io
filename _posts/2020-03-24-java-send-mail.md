---
layout: post
title: JAVA 메일 발송 예제
date: 2020-03-24
tags: JAVA
---

친구가 자바 메일발송을 물어봤는데 해본적이 없어서...
구글, 네이버 메일발송 테스트 해보고 포스팅함

구글, 네이버가 발송하는 방법이 약간 다르기 때문에 두가지 방법 모두 작성

javax.mail 라이브러리 1.6.2버전 사용
라이브러리 다운로드 : [https://mvnrepository.com/artifact/com.sun.mail/javax.mail/1.6.2](https://mvnrepository.com/artifact/com.sun.mail/javax.mail/1.6.2)   

## 구글 메일 발송

먼저 메일 발송할 구글 계정으로 로그인한 후
계정 - 보안 - 보안수준이 낮은 앱의액세스 항목을 사용으로 바꿔준다

![img]({{ '/assets/images/java-send-mail/google1.PNG' | relative_url }}){: .center-image }
![img]({{ '/assets/images/java-send-mail/google2.PNG' | relative_url }}){: .center-image }

그후 메일로 보안 수준이 낮은 앱에 대한 엑세스가 허용됬다는 메일이 오는데
메일을 클릭해 본인 활동임을 확인해주고

![img]({{ '/assets/images/java-send-mail/google3.PNG' | relative_url }}){: .center-image }

자바 소스를 이용해 메일을 발송해준다

{% highlight java %}
import java.util.Properties;

import javax.activation.DataHandler;
import javax.activation.DataSource;
import javax.activation.FileDataSource;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.PasswordAuthentication;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeMultipart;


public class MailTest {

    public static void main(String[] args) {
		
	//SMTP 서버 정보 설정
        Properties props = new Properties();
        
        props.put("mail.smtp.starttls.enable", "true");        
        props.put("mail.smtp.host", "smtp.gmail.com"); //발송 STMP 서버
        props.put("mail.smtp.port", "587"); //SMTP서버의 포트   
        props.put("mail.smtp.auth", "true");
        
        //SMTP 서버 정보와 사용자 계정를 기반으로 Session 클래스의 인스턴스를 생성
        final String user   = "test@gmail.com"; //메일을 발송할 유저의 계정
        final String password  = "password"; //메일을 발송할 유저의 패스워드
        
        String to = "test@naver.com"; //메일을 받을 계정
        
        Session session = Session.getInstance(props, new javax.mail.Authenticator() {
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(user, password);
            }
        });
        
        try {
	    MimeMultipart multipart = new MimeMultipart();

            //mime type을 html로 설정 후 내용을 입력한다
	    MimeBodyPart mbp = new MimeBodyPart(); 
            mbp.setText("<a href=\"http://www.google.com\">구글</a>"  //하이퍼링크 테스트
            	      + "<div style=\"color:red\"> css test </div>", //css 테스트
            		  "UTF-8", "html"); 			     //인코딩을 설정해주지 않으면 한글이 깨진다
            
            multipart.addBodyPart(mbp); //Multi-part에 bodyPart를 추가
            
            //이미지 첨부
            MimeBodyPart mbp2 = new MimeBodyPart();
            DataSource dataSource = new FileDataSource("imageTest.jpg"); //첨부할 파일
            DataHandler dataHandler = new DataHandler(dataSource);
            mbp2.setDataHandler(dataHandler);
            mbp2.setFileName("테스트.jpg"); //첨부할 파일 이름

            multipart.addBodyPart(mbp2); //Multi-part에 bodyPart2를 추가
            
            
            //메일 발신자와 수신자, 제목 그리고 내용 작성을 위한 MimeMessage객체 생성
            Message message = new MimeMessage(session); 
            
	    //Content에 multipart를 설정해줌
            message.setContent(multipart);
            
            //수신자 설정
            message.addRecipient(Message.RecipientType.TO, new InternetAddress(to));
            
            //제목 설정
            message.setSubject("한글 테스트");
            
            //이메일 보내기
            Transport.send(message);
            System.out.println("send mail success.");
            
        } 
        catch (MessagingException e) {            
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

## 네이버 메일 발송

네이버 로그인 후
메일 - 환경설정 - POP3/IMAP 설정 - IMAP/SMTP 설정에서
IMAP/SMTP 사용을 사용함으로 체크
![img]({{ '/assets/images/java-send-mail/naver1.PNG' | relative_url }}){: .center-image }
![img]({{ '/assets/images/java-send-mail/naver2.PNG' | relative_url }}){: .center-image }
그 후에 자바에서 메일발송을 한다

{% highlight java %}
import java.util.Properties;

import javax.activation.DataHandler;
import javax.activation.DataSource;
import javax.activation.FileDataSource;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.PasswordAuthentication;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeMultipart;


public class MailTest {

    public static void main(String[] args) {
		
	//SMTP 서버 정보 설정
        Properties props = new Properties();
        
        props.put("mail.smtp.host", "smtp.naver.com"); //발송 STMP 서버
        props.put("mail.smtp.port", "465"); //SMTP서버의 포트   
        props.put("mail.smtp.auth", "true");

        //SSL 활성화
        props.put("mail.smtp.ssl.enable", "true"); 
        props.put("mail.smtp.ssl.trust", "smtp.naver.com");
        props.put("mail.smtp.socketFactory.port", "465");
        props.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
        
        //SMTP 서버 정보와 사용자 계정를 기반으로 Session 클래스의 인스턴스를 생성
        final String user   = "test@naver.com";
        final String password  = "password";
        
        String to = "test@naver.com";
          
        Session session = Session.getInstance(props, new javax.mail.Authenticator() {
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(user, password);
            }
        });
        
        try {
            MimeMultipart multipart = new MimeMultipart();

            //mime type을 html로 설정 후 내용을 입력한다
	    MimeBodyPart mbp = new MimeBodyPart(); 
            mbp.setText("<a href=\"http://www.naver.com\">네이버</a>" //하이퍼링크 테스트
            	      + "<div style=\"color:red\"> css test </div>", //css 테스트
            		  "UTF-8", "html"); 			     //인코딩을 설정해주지 않으면 한글이 깨진다
            
            multipart.addBodyPart(mbp); //Multi-part에 bodyPart를 추가
            
            //이미지 첨부
            MimeBodyPart mbp2 = new MimeBodyPart();
            DataSource dataSource = new FileDataSource("imageTest.jpg"); //첨부할 파일
            DataHandler dataHandler = new DataHandler(dataSource);
            mbp2.setDataHandler(dataHandler);
            mbp2.setFileName("테스트.jpg"); //첨부할 파일 이름

            multipart.addBodyPart(mbp2); //Multi-part에 bodyPart2를 추가
            
            
            //메일 발신자와 수신자, 제목 그리고 내용 작성을 위한 MimeMessage객체 생성
            Message message = new MimeMessage(session); 
            
	    //Content에 multipart를 설정해줌
            message.setContent(multipart);
            
	    //발신자 설정
            message.setFrom(new InternetAddress(user));

            //수신자 설정
            message.addRecipient(Message.RecipientType.TO, new InternetAddress(to));
            
            //제목 설정
            message.setSubject("한글 테스트");
            
            //이메일 보내기
            Transport.send(message);
            System.out.println("send mail success.");
            
        } 
        catch (MessagingException e) {            
            e.printStackTrace();
        }
    }
}
{% endhighlight %}