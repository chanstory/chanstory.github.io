---
layout: post
title: POI라이브러리 word 파일 파싱, 생성
date: 2020-01-08 15:30
tags: java
---

word파일 파싱할 일이 있어 POI 라이브러리를 사용한 word파일 파싱, 생성 예제 정리
라이브러리는 3.9버전 사용

자세한 사용법은 [https://www.tutorialspoint.com/apache_poi_word/apache_poi_word_quick_guide.htm](https://www.tutorialspoint.com/apache_poi_word/apache_poi_word_quick_guide.htm) 참조하면 된다.

word97~2003 파일 파싱(확장자 .doc)
{% highlight scss %}
public String DocFileContentParser(String filePathName) {
    FileInputStream input = null;

    try {
	input = new FileInputStream(filePathName);//ex. src/resources/parsingTest.doc
	POIFSFileSystem fs = new POIFSFileSystem(input);
   
	HWPFDocument doc = new HWPFDocument(fs);
	WordExtractor we = new WordExtractor(doc);
    
	input.close();
	return we.getText();
 
    } catch (Exception e) {
	e.printStackTrace();
    }finally {
	try {
	    input.close();
	} catch (IOException e) {
	    e.printStackTrace();
	}
    }
	
    return "";
}
{% endhighlight %}


word파일 파싱(확장자 .docx)
{% highlight scss %}
public String DocxFileContentParser(String filePathName) {
    FileInputStream input = null;

    try {
	input = new FileInputStream(filePathName);

	XWPFDocument docx = new XWPFDocument(OPCPackage.open(input));
	XWPFWordExtractor extractor = new XWPFWordExtractor(docx);

	input.close();
	return extractor.getText();
    } catch(Exception e) {
	e.printStackTrace();
    }finally {
	try {
	    input.close();
	} catch (IOException e) {
	    e.printStackTrace();
	}
    }
    return "";
}
{% endhighlight %}

word파일 생성
{% highlight scss %}
public void DocFileContentWrite(String filePathName, String text) {
		
    FileOutputStream out = null;
    try {
	XWPFDocument document = new XWPFDocument();
	out = new FileOutputStream( new File(filePathName));//ex. src/resources/result.docx
	
	XWPFParagraph paragraph = document.createParagraph();
	XWPFRun run = paragraph.createRun();
	run.setText(text);
	
	document.write(out);
	out.close();
    } catch (Exception e) {
	e.printStackTrace();
    }finally {
	try {
	    out.close();
	} catch (IOException e) {
	    e.printStackTrace();
	}
    }
}
{% endhighlight %}

.doc 파일의 생성예제는 찾지못했다..
HWPFDocument 객체를 생성할때 POIFSFileSystem을 파라미터로 받는데
POIFSFileSystem을 생성할 때 FileInputStream을 받지않는 기본 생성자가 없어
생성자체를 못하는게 아닌가 추측된다..