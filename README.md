# Ansible Loops

This repository uses a simple Vagrantfile and a basic playbook to
illustrate the use of some of the types of loops available inside Ansible.

## Getting started

### Install required tools

1. [Ansible](http://docs.ansible.com/ansible/intro_installation.html)
2. [Vagrant](https://www.vagrantup.com/downloads.html)

### Get and run the code

1. `git clone git@github.com:ctorgalson/ansible-loops.git`
2. `cd ansible-loops`
3. `vagrant up`

## The example tasks

The included playbook contains a series of examples meant to illustrate
one of [Ansible's loop constructs](http://docs.ansible.com/ansible/playbooks_loops.html)
or some related concept. The actual tasks performed are mostly arbitrary, but
are directly applicable to the day-to-day use of Ansible.

### Loop 1: create users using `with_items`.

The first example uses the `with_items` loop to ensure that two users are
present on the target system:

```yaml
# Variables for Loop 1.
users_with_items:
  - name: "alice"
    personal_directories:
      - "bob"
      - "carol"
      - "dan"
  - name: "bob"
    personal_directories:
      - "alice"
      - "carol"
      - "dan"
```

```yaml
- name: "Loop 1: create users using 'with_items'."
  user:
    name: "{{ item.name }}"
  with_items: "{{ users_with_items }}"
```

### Loop 2: create common users' directories using `with_nested`.

The second example builds on Loop 1 to ensure that each of the users' home
directories contain two additional folders:

```yaml
# Variables for Loop 2 (also uses `users_with_items` from Loop 1).
common_directories:
  - ".ssh"
  - "loops"
```

```yaml
- name: "Loop 2: create common users' directories using 'with_nested'."
  file:
    dest: "/home/{{ item.0.name }}/{{ item.1 }}"
    owner: "{{ item.0.name }}"
    group: "{{ item.0.name }}"
    state: directory
  with_nested:
    - "{{ users_with_items }}"
    - "{{ common_directories }}"
```

### Loop 3: create personal users' directories using `with_subelements`.

The third example re-uses the `users_with_items` variable from Loop 1 to
create a unique set of directories in each user's home directory using
`with_subelements`:

```yaml
- name: "Loop 3: create personal users' directories using 'with_subelements'."
  file:
    dest: "/home/{{ item.0.name }}/{{ item.1 }}"
    owner: "{{ item.0.name }}"
    group: "{{ item.0.name }}"
    state: directory
  with_subelements:
    - "{{ users_with_items }}"
    - personal_directories
```

### Loop 4: create users using 'with_dict'.

The fourth example show how to use the `with_dict` type of loop:

```yaml
# Variables for Loop 4 (also uses `common_directories` from Loop 2).
users_with_dict:
  carol:
    common_directories: "{{ common_directories }}"
  dan:
    common_directories: "{{ common_directories }}"
```

```yaml
- name: "Loop 4: create users using 'with_dict'."
  user:
    name: "{{ item.key }}"
  with_dict: "{{ users_with_dict }}"
```

### Loop 5: create personal directories if they don't exist.

The fifth example shows one of the limitations of the `with_dict` loop
from Loop 4, how to register a variable and loop over it, and how to use
a jinja2 filter to transform some existing variables into a more usable
form:

```yaml
- name: "Get list of extant users."
  args:
    chdir: "/home"
  shell: "find * -type d -prune -not -path vagrant | sort"
  register: "home_directories"
  changed_when: false

- name: "Loop 5: create personal user directories if they don't exist."
  file:
    dest: "/home/{{ item.0 }}/{{ item.1 }}"
    owner: "{{ item.0 }}"
    group: "{{ item.0 }}"
    state: directory
  with_nested:
    - "{{ home_directories.stdout_lines }}"
    - "{{ home_directories.stdout_lines | union(common_directories) }}"
  when: "'{{ item.0 }}' != '{{ item.1 }}'"
```

### Loop 6: list user directory contents and verify playbook results.

The final loop shows another example of how to iterate over a registered
variable, as well as another way of using jinja2 filters to make debug
output easier to read.

```yaml
- name: "Loop 6: list user directory contents."
  args:
    chdir: "/home/{{ item }}"
  shell: "pwd && find -type d | sort"
  register: directory_contents
  with_items: "{{ home_directories.stdout_lines }}"
  changed_when: false

- name: "Verify playbook results."
  debug:
    msg: "{{ directory_contents.results | map(attribute='stdout_lines') | list }}"
```
