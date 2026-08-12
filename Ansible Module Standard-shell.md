# Ansible Module Standard — shell

## 1. 적용 범위

Ansible Playbook에서 `ansible.builtin.shell` Module을 사용하는 Task에 적용한다.

---

## 2. 사용 원칙

* Ansible 전용 Module로 구현 가능한 작업은 해당 Module을 우선 사용한다.
* OS 명령 또는 기존 Shell Script 실행이 필요한 경우 `ansible.builtin.shell`을 사용한다.
* `ansible.builtin.command`는 사용하지 않는다.

---

## 3. 기본 작성 형식

모든 Shell Task는 `cmd:` 형식으로 작성한다.

```yaml
- name: <Task description>
  ansible.builtin.shell:
    cmd: "<command>"
```

---

## 4. 명령 작성 규칙

### 4.1 단일 명령

한 줄로 작성한다.

```yaml
- name: Check hostname
  ansible.builtin.shell:
    cmd: "hostname"
```

### 4.2 여러 명령

여러 개의 독립된 명령을 실행하는 경우 `|`를 사용한다.

```yaml
- name: Configure Oracle
  ansible.builtin.shell:
    cmd: |
      mkdir -p /u01/app/oracle
      chown oracle:oinstall /u01/app/oracle
      chmod 775 /u01/app/oracle
```

### 4.3 하나의 명령을 여러 줄로 표현

하나의 명령을 가독성을 위해 여러 줄로 분리하는 경우 `\`로 연결한다.

```yaml
- name: Run Grid Setup
  ansible.builtin.shell:
    cmd: |
      ./gridSetup.sh \
        -silent \
        -responseFile /stage/grid.rsp \
        -waitforcompletion
```

* 연결되는 행의 끝에 `\`를 사용한다.
* 마지막 행에는 `\`를 사용하지 않는다.
* 독립된 여러 명령에는 `\`를 사용하지 않는다.

### 4.4 YAML Block 표기

Shell multiline은 `|`로 통일한다.

* `|` : 사용
* `>` : 사용하지 않는다. YAML에서 줄바꿈이 공백으로 변환되어 Shell 명령의 줄 구조가 변경된다.
* `>-` : 사용하지 않는다. `>`와 동일하게 줄바꿈이 공백으로 변환되고 마지막 줄바꿈도 제거된다.
* `|-` : 사용하지 않는다. 마지막 줄바꿈이 제거되어 표준 형식을 일관되게 유지할 수 없다.

---

## 5. changed_when

### 5.1 조회/검증 명령

조회 또는 검증만 수행하는 Shell Task에는 `changed_when: false`를 사용한다.

```yaml
- name: Check Oracle CRS
  ansible.builtin.shell:
    cmd: "crsctl check crs"
  register: crs_status
  changed_when: false
```

### 5.2 상태 변경 명령

상태 변경을 목적으로 하는 Shell Task에는 `changed_when`을 사용하지 않는다.

```yaml
- name: Run Grid Setup
  ansible.builtin.shell:
    cmd: |
      ./gridSetup.sh \
        -silent \
        -responseFile /stage/grid.rsp
```

`changed_when: true` 또는 `changed_when: false`를 임의로 지정하지 않는다.

---

## 6. failed_when

`failed_when`은 기본적으로 사용하지 않는다.

각 Shell 명령의 return code를 분석하여 개별적으로 `failed_when` 조건을 작성하지 않는다. Ansible의 기본 return code 판정을 사용한다.

### 예외

조회/검증 명령에서 `rc != 0`이 오류가 아닌 정상적인 조회 결과임이 명확한 경우에만 `failed_when: false`를 사용한다.

```yaml
- name: Check process
  ansible.builtin.shell:
    cmd: "pgrep -f ora_pmon"
  register: result
  changed_when: false
  failed_when: false
```

그 외에는 `failed_when`을 추가하지 않는다.

`failed_when: true`는 사용하지 않는다.
**명령 실행 결과와 관계없이 항상 Task를 실패 처리하므로 사용하지 않는다.**

---

## 7. ignore_errors

명령 실패를 표시하면서도 후속 Task를 계속 수행해야 하는 경우 `ignore_errors: true`를 사용한다.

```yaml
- name: Run Oracle Installer
  ansible.builtin.shell:
    cmd: |
      ./runInstaller.sh \
        -silent \
        -responseFile /stage/db.rsp
  register: installer_result
  ignore_errors: true
```

### 규칙

* 오류를 숨기기 위한 목적으로 사용하지 않는다.
* `ignore_errors: true` 사용 시 `register`를 사용한다.
* Oracle Grid/DB 설치와 같이 중간 오류 후에도 후속 단계 진행이 필요한 경우 사용할 수 있다.
* 예상하지 못한 오류를 `ignore_errors`로 무시하지 않는다.
* 파일 미존재, 경로 오류 등 사전 조건 문제는 선행 Task에서 확인한다.

---

## 8. 표준 Template

### 8.1 단일 명령

```yaml
- name: <Task description>
  ansible.builtin.shell:
    cmd: "<command>"
```

### 8.2 여러 명령

```yaml
- name: <Task description>
  ansible.builtin.shell:
    cmd: |
      <command 1>
      <command 2>
      <command 3>
```

### 8.3 긴 단일 명령

```yaml
- name: <Task description>
  ansible.builtin.shell:
    cmd: |
      <command> \
        <option> \
        <option>
```

### 8.4 조회/검증

```yaml
- name: Check <status>
  ansible.builtin.shell:
    cmd: "<command>"
  register: <result>
  changed_when: false
```

### 8.5 오류 발생 후 계속 진행

```yaml
- name: <Task description>
  ansible.builtin.shell:
    cmd: |
      <command>
  register: <result>
  ignore_errors: true
```
