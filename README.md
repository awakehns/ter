# Домашнее задание к занятию "`Введение в Terraform`" - `Демин Герман`

### Задание 1

`terraform --version`

Версия Terraform:

![ter-ver](/img/ter-ver.png)

`git clone https://github.com/netology-code/ter-homeworks.git`

Склонированный репозиторий:

![ter-clone](/img/ter-clone.png)

#### 1.

`terraform init`

Скачанные зависимости:

![ter-init](/img/ter-init.png)

#### 2.

По файлу .gitignore, единственный Terraform-файл, в котором допустимо сохранить личную, секретную информацию (логины, пароли, ключи, токены и т.д.), это файл с именем .terraformrc.

#### 3.

`terraform apply --auto-approve`

Применение кода:

![ter-apply](/img/ter-apply.png)

Строчка со значением random_password:
`"result": "Dy7e1bDWzm0FzpqE",`

#### 4.

Раскомментированный блок кода:

```
resource "docker_image" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "1nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string_FAKE.resulT}"

  ports {
    internal = 80
    external = 9090
  }
}
```

`terraform validate`

Ошибки:

![ter-error](/img/ter-error.png)

Объяснение ошибок:

Terraform выдал две четкие ошибки:

`Error: Missing name for resource для resource "docker_image".`

Каждый блок resource в Terraform должен иметь два "лейбла" (метки): тип ресурса и имя ресурса. В блоке resource "docker_image" отсутствует имя. Terraform использует это имя для уникальной идентификации данного экземпляра ресурса конфигурации.

`Error: Invalid resource name для resource "docker_container" "1nginx".`

Имя ресурса (в данном случае "1nginx") должно начинаться с буквы или нижнего подчеркивания (_). Оно не может начинаться с цифры. Это правило именования, принятое в Terraform, чтобы избежать путаницы и обеспечить корректную работу парсера.

Также допущено две ошибки в:
`name  = "example_${random_password.random_string_FAKE.resulT}"`
result должен писаться только нижним регистром, а объявленный выше ресурс называется без приписки "_FAKE"

Исправленный блок кода:
```
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx_container" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 9090
  }
}
```

#### 5.

`terraform validate`

`terraform apply --auto-approve`

`docker ps`

Применение нового кода:

![ter-new-1](/img/ter-new-1.png)
![ter-new-2](/img/ter-new-2.png)

docker ps:

![docker-ps-1](/img/docker-ps-1.png)

#### 6.

Изменение имени контейнера на hello_world

```
resource "docker_container" "nginx_container" {
  image = docker_image.nginx.image_id
  name  = "hello_world"

  ports {
    internal = 80
    external = 9090
  }
}
```

`terraform apply --auto-approve`

В чём опасность применения ключа -auto-approve?

Ключ -auto-approve автоматически подтверждает все изменения, предложенные Terraform, без запроса у пользователя. Главная опасность в потере контроля: нет возможности увидеть и одобрить план, что может привести к случайному удалению важных ресурсов, непредвиденным изменениям или высоким расходам из-за ошибок в коде. Это снижает безопасность и контроль над инфраструктурой.

Зачем может пригодиться данный ключ?

Ключ -auto-approve незаменим для автоматизации, особенно в конвейерах CI/CD, где ручное подтверждение невозможно. Он позволяет создавать и уничтожать инфраструктуру в скриптах, ускоряя тестирование и развёртывание. Это удобно для пакетных операций или когда план изменений уже многократно проверен и утверждён.

`docker ps`

docker ps:

![docker-ps-2](/img/docker-ps-2.png)

#### 7.

`terraform destroy --auto-approve`

`cat ./terraform.tfstate`

```
{
  "version": 4,
  "terraform_version": "1.12.1",
  "serial": 13,
  "lineage": "474aa831-ec2f-ef04-6513-5bbc87b84292",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```

Удаление ресурсов:

![ter-destroy](/img/ter-destroy.png)

Образ не удалился после применения 
`terraform destroy --auto-approve`
потому что явно указан параметр
`keep_locally = true`
в main.tf. Он говорит терраформу, что образы не должны удалятся с локальной машины при применении команды
`terraform destroy`

подтверждение из документации:

![ter-doc](/img/ter-doc.png)
