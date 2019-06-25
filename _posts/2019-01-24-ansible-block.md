---
---
### 2019-01-24

Today I learned about Ansible's `block`. I was trying to group some task together and to avoid executing them with a `when` statement.

I came across [this post](https://stackoverflow.com/a/41561885) on stackoverflow which was exactly what I wanted to do.
{% raw %}
```
---

- name: Check for already install app
  stat:
    path: /opt/app/bin/MyApp
  register: app_installed

- block:
    - name: Fetch and extract App
      unarchive:
        src: http://someplace.com/getApp/app.zip
        dest: /opt/
        remote_src: yes

    - name: Install Application
      shell: /opt/app/installer.sh
      args:
        executable: /bin/bash

    - name: Application Install Summary
      vars:
        msg: |
          The application is now installed. This is fake, but it would print only once.

          The block can use a when to disable those tasks.

      debug:
        msg: "{{ msg.split('\n') }}"

  when: app_installed.stat.exists == False
```
{% endraw %}
