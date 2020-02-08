---
layout: post
title: Spring Boot, Spring Data JPA 다중 DB 설정
date: 2020-02-09
tags: SpringBoot JPA
---

며칠전에 갑자기 스프링부트에서 다중DB 설정하는 법이 궁금해서 찾아봤다.
AWS서버에 올라가있는 MySQL과 로컬에 H2 DB 사용

H2 DB  설정

{% highlight scss %}
package com.myapp.dataSource;

import java.util.HashMap;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
@PropertySource({ "classpath:application.properties" })
@EnableJpaRepositories( 
	//Repository가 있는 패키지
	basePackages = "com.myapp.repository.h2Repository",
	//밑에서 생성할 EntityManagerFactory
	entityManagerFactoryRef = "h2EntityManager", 
	//밑에서 생성할 트랜잭션 매니저
	transactionManagerRef = "h2TransactionManager"
)
public class H2Config {
	
	//
	@Autowired
	private Environment env;

	@Bean
	//MySQL 설정도 같은 객체들을 사용하기 떄문에 객체명으로 DI 될 때 우선적으로 사용해주기 위해
	@Primary
	public LocalContainerEntityManagerFactoryBean h2EntityManager() {
		LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
		em.setDataSource(h2DataSource());
		//사용될 Entity들의 패키지
		em.setPackagesToScan(new String[] { "com.myapp.h2Object" });

		HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
		em.setJpaVendorAdapter(vendorAdapter);
		HashMap<String, Object> properties = new HashMap<>();
		properties.put("hibernate.hbm2ddl.auto", env.getProperty("spring.jpa.hibernate.ddl-auto"));
		properties.put("hibernate.dialect", env.getProperty("datasource.h2.dialect"));
		em.setJpaPropertyMap(properties);

		return em;

	}
	
	@Bean
	@Primary
	//dataSource 생성
	public DataSource h2DataSource() {
		
		DriverManagerDataSource dataSource = new DriverManagerDataSource();
		dataSource.setDriverClassName(env.getProperty("datasource.h2.driverClassName"));
		dataSource.setUrl(env.getProperty("datasource.h2.url"));
		dataSource.setUsername(env.getProperty("datasource.h2.username"));
		dataSource.setPassword(env.getProperty("datasource.h2.password"));
		System.out.println("h2 data Source url : " + dataSource.getUrl() + ", userName : " + dataSource.getUsername() + ", password : " + dataSource.getPassword());
		return dataSource;
	}

	@Bean
	@Primary
	//트랜잭션 매니저 생성
	public PlatformTransactionManager h2TransactionManager() {

		JpaTransactionManager transactionManager = new JpaTransactionManager();
		transactionManager.setEntityManagerFactory(h2EntityManager().getObject());
		return transactionManager;
	}

}
{% endhighlight %}

H2 DB를 사용한 레파지토리

{% highlight scss %}
//이 패키지 이름이 설정에 @EnableJpaRepositories 어노테이션에 명시되어야 함
package com.myapp.repository.h2Repository;


import org.springframework.data.jpa.repository.JpaRepository;

import com.myapp.h2Object.Member;

public interface MemberRepository extends JpaRepository<Member, Long>
{

}
{% endhighlight %}

properties파일 관련내용

{% highlight scss %}
#dataSource
datasource.mysql.driverClassName=com.mysql.jdbc.Driver
datasource.mysql.url=DBUrl
datasource.mysql.username=userName
datasource.mysql.password=password
datasource.mysql.dialect=org.hibernate.dialect.MySQL5InnoDBDialect

datasource.h2.driverClassName=org.h2.Driver
datasource.h2.url=jdbc:h2:tcp://localhost/~/test
datasource.h2.username=sa
datasource.h2.password=
datasource.h2.dialect=org.hibernate.dialect.H2Dialect
{% endhighlight %}

이제 H2Config에서 설정해준 com.myapp.h2Object 패키지안에 Entity들을 만들어 사용하면된다!

Mysql도 똑같이 설정하되 설정파일에 @Primary 어노테이션은 제거해야함!