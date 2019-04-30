[![Build Status](https://travis-ci.com/Tennki/ansible-role-db.svg?branch=master)](hhttps://travis-ci.com/Tennki/ansible-role-db)

Ansible-db-role
=========
Тестовая роль db

Краткое руководство по настройке Travis + Molecule + GCE
# Генерируем ключ для подключения по SSH
ssh-keygen -t rsa -f google_compute_engine -C 'travis' -q -N ''
# Создаем ключ в метадате проекта infra в GCP

# Должен быть предварительно создан сервисный аккаунт и скачаны креды (credentials.json)
Сервисному аккаунту надо добавить роли:
 Администратор Compute
 Пользователь сервисного аккаунта

# Должна быть предварительно подключена данная репа в тревисе
travis encrypt GCE_SERVICE_ACCOUNT_EMAIL='travis-ci@infra-######.iam.gserviceaccount.com' --add
travis encrypt GCE_CREDENTIALS_FILE="\"\$(pwd)/credentials.json\"" --add
travis encrypt GCE_PROJECT_ID='infra-######' --add

# шифруем файлы
tar cvf secrets.tar credentials.json google_compute_engine
travis login
travis encrypt-file secrets.tar --add

# пушим и проверяем изменения
git commit -m 'Added Travis integration'
git push

# Если не использовать travis encrypt-file для шифрования архива или использовать другую версию openssl, то могут возникнуть проблемы при расшифровке на стороне тревиса. На стороне travis установлена версия 1.0.1f.
# Можно обойти так. В настройках репозитория создаем ключ travis_key.
- шифруем у себя: 
docker run --rm -it -v $(pwd):/export frapsoft/openssl aes-256-cbc -k $travis_key -in /export/try_secrets.tar.enc -out /export/secrets.tar
- добавляем в travis.yml
docker run --rm -it -v $(pwd):/export frapsoft/openssl aes-256-cbc -d -k $travis_key -in /export/try_secrets.tar.enc -out /export/secrets.tar
