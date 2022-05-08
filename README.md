
# _CloudFormation файл конфигурации_

---

## _Prerequisites:_

- An existing Amazon Route 53 hosted zone

---

## _Parameters:_

_DomainName:_
    
    Type: String
    Description: The DNS name of an existing Amazon Route 53 hosted zone

_BucketName:_

    Type: String
    Description: Amazon S3 files bucket name

_HostedZoneId:_
    
    Type: String
    Description: Amazon Route53 Hosted Zone Id

_CertificateArn:_

    Type: String
    Description: Certificate Link
---

## _Created Resources:_
- S3 bucket
- S3 policy
- CloudFront distribution for S3 files bucket
- Route53 A-record to the CloudFront files distribution (RecordSet)

Also, a CloudFront origin access identity is created. It is needed to create a distribution.

---
---

# Всё необходимое для запуска данной системы:

## _Подготовим сертификат_

	1. Заходим в AWS Консоль -> AWS Certificate Manager.
	2. Выбираем N. Virginia (us-east-1) регион, всё должно быть тут запущено для простоты.
        А-запись в Route 53 к CloudFront distribution, что пытается найти сертификат может работать только в этом регионе.
        Это потому, что доменное связывание - это вещь глобальная.
	3. Жмём "Request certificate" и заказываем сертификат для своего домена.
	4. ACM даёт нам cname запись, копируем её в DNS в Route 53 чтоб сертификат провалидировался
        (можно просто нажать "Create records in Route 53").. Валидация занимает время ..
	5. Копируем Amazone Resource Name сертификата. Это значение для переменной CertificateArn.

## _Запуск_

1. Выбираем предпочтительный формат файла (.json or .yaml). Оба описывают одинаковую конфигурацию.
2. Переходим в CloudFormation
3. Жмём "Create stack" и начинаем создавать стек
4. Жмём "Upload a template file" и грузим туда наш файл конфигурации
5. Жмём "Next"
6. Вводим значения, пример формата рабочих входных данных:
    
    - Stack name - bucket-policy-oai-distribution-record-set
    - BucketName - sandro.click
    - CertificateArn - arn:aws:acm:us-east-1:333051377810:certificate/4e993561-6ecc-4218-9377-c248890afb3d
    - DomainName - sandro.click
    - HostedZoneId - Z05737403RELK40MGJ5GN


    Можно было бы обойтись без параметра BucketName и вместо него использовать DomainName. 
    То же самое с HostedZoneId, при создании RecordSet можно использовать HostedZoneName, т.е. опять же DomainName
    HostedZoneId у CloudFront distribution всегда = Z2FDTNDATAQYW2 и это используется в шаблоне, входная переменная не нужна, т.к. это константа.
    Не могу не написать об этом, т.к. это возможность оптимизировать шаблон уменьшив количество входных данных.
    Так же название bucket должно быть глобально уникальным и его совпадение с доменом это обеспечивает.

Если какие-то из значений невалидны, то ошибка будет выдана при создании стека.

7. Жмём "Next"
8. Если есть желание или необходимость, то можно добавить тегов к стеку и другие опциональные штуки. Жмём "Next"
9. Перепроверяем введенный данные.  Жмём "Next"
10. Теперь следим за иветтами
11. Как только создастся ресурс S3B225ZM, то можно перейти в S3 и добавить index.html в созданную папку. Это нужно будет
    для проверки, что всё работает. Перед удалением стека папку необходимо будет очистить
12. Ждём пока всё остальное создастся, это займёт пару минут
13. Открываем домен в браузере! Всё работает!

Если же что-то не работает, то необходимо проверить наличие index.html в bucket и наличие каждого из создаваемых ресурсов. Возможно какой-то из них ещё находится в стадии создания

- В Parameters можно глянуть параметры
- В Outputs находится CloudFront Distribution ID
- В Template можно глянуть шаблон в дизайнере

-------------------------------------------------------------------------------------------------------------------------------------------
