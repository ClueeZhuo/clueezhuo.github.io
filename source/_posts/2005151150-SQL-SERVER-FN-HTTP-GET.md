---
title: 2005151150@SQL-SERVER-FN-HTTP-GET
date: 2020-05-15 11:51:20
categories:
- SQL
tags:
- MSSQL
---
```sql
SET QUOTED_IDENTIFIER ON
SET ANSI_NULLS ON
GO
CREATE FUNCTION FN_HTTP_GET(
@URL VARCHAR(256),
@DATA VARCHAR(2000),
@REQ_H_ACCEPT VARCHAR(256),
@REQ_H_CONTENT_TYPE VARCHAR(256)
)
RETURNS VARCHAR(5000)
AS 
BEGIN

DECLARE 
@object int,
@returnStatus int,
@returnText varchar(5000),
@errMsg varchar(2000),
@httpStatus varchar(20);



/* 初始化 */  
    EXEC @returnStatus = SP_OACreate 'Msxml2.ServerXMLHTTP.3.0',@object OUT;  
    
    IF @returnStatus <> 0  
    BEGIN  
EXEC SP_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
RETURN ('初始化对象失败，' + @errMsg + ISNULL(@returnText,''));  
    END  
    
    
    
/*创建链接*/
EXEC @returnStatus= SP_OAMethod @object,'open',NULL,'get',@URL,'false';

IF @returnStatus <> 0
BEGIN
EXEC SP_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
RETURN ('创建连接失败，' + @errMsg + ISNULL(@returnText, ''));
END

EXEC @returnStatus=SP_OAMethod @object,'setRequestHeader',NULL,'Accept',@REQ_H_ACCEPT;
EXEC @returnStatus=SP_OAMethod @object,'setRequestHeader',NULL,'Content-Type',@REQ_H_CONTENT_TYPE;
EXEC @returnStatus=SP_OAMethod @object,'setRequestHeader',NULL,'Content-Length','1000000';



/*发起请求*/
EXEC @returnStatus= SP_OAMethod @object,'send',NULL,@DATA;
IF @returnStatus <> 0 
BEGIN 
EXEC SP_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
RETURN ('发起请求失败，' + @errMsg + ISNULL(@returnText, ''));
END



/*获取HTTP状态代码*/
EXEC @returnStatus = SP_OAGetProperty @Object, 'Status', @httpStatus OUT;

    IF @returnStatus <> 0
    BEGIN
        EXEC sp_OAGetErrorInfo @Object, @errMsg OUT, @returnText OUT;
        RETURN ('获取HTTP状态代码失败，' + @errMsg + ISNULL(@returnText, ''));
    END

    IF @httpStatus <> 200
    BEGIN
        RETURN ('访问错误，HTTP状态代码:' + @httpStatus);
    END
        
    
        
/*获取返回信息*/
EXEC @returnStatus= SP_OAGetProperty @object,'responseText',@returnText OUT;

IF @returnStatus <> 0 
BEGIN 
EXEC SP_OAGetErrorInfo @object, @errMsg OUT, @returnText OUT;
RETURN ('获取返回信息失败，' + @errMsg + ISNULL(@returnText, ''));
END

 
RETURN @returnText;
END 
;
GO

```
---
```sql
DECLARE @Response NVARCHAR(MAX);
SELECT @Response
    = [dbo].[FN_HTTP_GET](
                             'http://xxx?keyword=' + @Account
                             + '&userType=2&companyCode=' + @CompanyCode
                             + '&rows=50&page=1&sidx=CreationTime+desc&sord=asc',
                             '',
                             '',
                             ''
                         );

SELECT @Response;

```