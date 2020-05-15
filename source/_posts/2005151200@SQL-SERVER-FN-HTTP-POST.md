---
title: 2005151200@SQL SERVER-FN_HTTP_POST
date: 2020-05-15 11:54:53
tags: SQL SERVER
---
```sql
SET QUOTED_IDENTIFIER ON
SET ANSI_NULLS ON
GO
-- [测试]
--sp_configure 'show advanced options', 1;
--GO
--RECONFIGURE;
--GO
--sp_configure 'Ole Automation Procedures', 1;
--GO
--RECONFIGURE;
--GO
--EXEC sp_configure 'Ole Automation Procedures';
--GO
CREATE FUNCTION FN_HTTP_POST
(
    @URL VARCHAR(256),
    @DATA VARCHAR(2000),
    @REQ_H_ACCEPT VARCHAR(256),
    @REQ_H_CONTENT_TYPE VARCHAR(256)
)
RETURNS VARCHAR(5000)
AS
BEGIN

    DECLARE @object INT,
            @returnStatus INT,
            @returnText VARCHAR(5000),
            @errMsg VARCHAR(2000),
            @httpStatus VARCHAR(20);



    /* 初始化 */
    EXEC @returnStatus = sp_OACreate 'Msxml2.ServerXMLHTTP.3.0', @object OUT;

    IF @returnStatus <> 0
    BEGIN
        EXEC sp_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
        RETURN ('初始化对象失败，' + @errMsg + ISNULL(@returnText, ''));
    END;



    /*创建链接*/
    EXEC @returnStatus = sp_OAMethod @object,
                                     'open',
                                     NULL,
                                     'post',
                                     @URL,
                                     'false';

    IF @returnStatus <> 0
    BEGIN
        EXEC sp_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
        RETURN ('创建连接失败，' + @errMsg + ISNULL(@returnText, ''));
    END;

    EXEC @returnStatus = sp_OAMethod @object,
                                     'setRequestHeader',
                                     NULL,
                                     'Accept',
                                     @REQ_H_ACCEPT;
    EXEC @returnStatus = sp_OAMethod @object,
                                     'setRequestHeader',
                                     NULL,
                                     'Content-Type',
                                     @REQ_H_CONTENT_TYPE;
    EXEC @returnStatus = sp_OAMethod @object,
                                     'setRequestHeader',
                                     NULL,
                                     'Content-Length',
                                     '1000000';



    /*发起请求*/
    EXEC @returnStatus = sp_OAMethod @object, 'send', NULL, @DATA;
    IF @returnStatus <> 0
    BEGIN
        EXEC sp_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
        RETURN ('发起请求失败，' + @errMsg + ISNULL(@returnText, ''));
    END;



    /*获取HTTP状态代码*/
    EXEC @returnStatus = sp_OAGetProperty @object, 'Status', @httpStatus OUT;

    IF @returnStatus <> 0
    BEGIN
        EXEC sp_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
        RETURN ('获取HTTP状态代码失败，' + @errMsg + ISNULL(@returnText, ''));
    END;

    IF @httpStatus <> 200
    BEGIN
        RETURN ('访问错误，HTTP状态代码:' + @httpStatus);
    END;



    /*获取返回信息*/
    EXEC @returnStatus = sp_OAGetProperty @object,
                                          'responseText',
                                          @returnText OUT;

    IF @returnStatus <> 0
    BEGIN
        EXEC sp_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
        RETURN ('获取返回信息失败，' + @errMsg + ISNULL(@returnText, ''));
    END;


    RETURN @returnText;
END;
GO

```
---
```sql
SELECT dbo.FN_HTTP_POST(
                           'http://www.svr.jiedu52.cn/Admin/AppUserWebApi/SubmitForm',
                           '{ "Account": "' + @Account + '", "Password": "123654", "Name": "' + @Name
                           + '", "State": "false", "IsShare": "false", "Remark": "auto", "UserType": "2", "CompanyCode": "'
                           + @CompanyCode + '" }',
                           'application/json',
                           'application/json'
                       );

WAITFOR DELAY '00:00:03:00';
```