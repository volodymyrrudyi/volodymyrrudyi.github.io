---
author: volodymyr
comments: true
date: 2015-04-16 00:56:00+00:00
layout: post
slug: spring-mvc-message-converter
title: Spring MVC CSV Message Converter
categories:
- Java
tags:
- Java
- Spring
---



Sometimes we need to return a CSV (yuk) document in the response. It’s possible to write the document directly to the output stream of the HTTP response but this approach is not generic and requires some boilerplate code if we need to use this functionality across several controllers/methods. In this short tutorial I’ll show how to use Spring MVC message converters mechanism to solve this task.

<!-- more -->


Let’s say we have a POJO:

    public class Country{

        private String name;
        private String capital;
        private long population;
        private double totalArea;

        // getters and setters
    }


And some controller method:

    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    public List<Country> getCountries()

Let’s try to use the wget tool and see what happens:

    wget --header=Accept:application/json -q  -O - http://localhost:8080/countries

__The result__:

    [
        {
            "name": "USA",
            "capital": "Washington",
            "population": 320206000,
            "totalArea": 9147593
        },
        {
            "name": "Ukraine",
            "capital": "Kyiv",
            "population": 44291413,
            "totalArea": 603500
        },
        {
            "name": "Japan",
            "capital": "Tokyo",
            "population": 126434964,
            "totalArea": 377944
        },
        {
            "name": "Mauritius",
            "capital": "Port Louis",
            "population": 1261208,
            "totalArea": 2040
        },
        {
            "name": "Montenegro",
            "capital": "Podgorica",
            "population": 620029,
            "totalArea": 13812
        }
    ]

But what if we need a CSV representation:

    wget --header=Accept:text/csv -q  -O - http://localhost:8080/countries


It will result in an empty response. Of course, Spring MVC doesn’t support CSV OOTB. But we can easily introduce it using Spring MVC message converters mechanism:

    @Override
    protected void writeInternal(Object object,
        HttpOutputMessage message)
    throws IOException, HttpMessageNotWritableException
    {

        message.getHeaders().set(HttpHeaders.CONTENT_TYPE, TEXT_CSV);

        try (final Writer writer = new OutputStreamWriter(message.getBody()))
        {
            final CsvBeanWriter beanWriter =
                new CsvBeanWriter(writer, CsvPreference.STANDARD_PREFERENCE);

            beanWriter.writeHeader(header);
            writeItems(beanWriter, object, header);
            beanWriter.flush();
        }
    }

Now let’s try again:  

    wget --header=Accept:text/csv -q  -O - http://localhost:8080/countries

And the new result:

    name,capital,population,totalArea
    USA,Washington,320206000,9147593.0
    Ukraine,Kyiv,44291413,603500.0
    Japan,Tokyo,126434964,377944.0
    Mauritius,Port Louis,1261208,2040.0
    Montenegro,Podgorica,620029,13812.0

 Now it works as expected. <a href="https://github.com/volodymyrrudyi/spring-message-converter-example">Download the example from the Github.</a>
