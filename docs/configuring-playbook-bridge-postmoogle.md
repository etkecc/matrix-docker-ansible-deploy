# Setting up Postmoogle email bridging (optional)

**Note**: email bridging can also happen via the [email2matrix](configuring-playbook-email2matrix.md) bridge supported by the playbook.

The playbook can install and configure [Postmoogle](https://github.com/etkecc/postmoogle) for you.

Postmoogle is a bridge you can use to have its bot user forward emails to Matrix rooms. It runs an SMTP email server and allows you to assign mailbox addresses to the rooms.

See the project's [documentation](https://github.com/etkecc/postmoogle) to learn what it does and why it might be useful to you.

## Prerequisites

Open the following ports on your server to be able to receive incoming emails:

  - `25/tcp`: SMTP
  - `587/tcp`: Submission (TLS-encrypted SMTP)

If you don't open these ports, you will still be able to send emails, but not receive any.

These port numbers are configurable via the `matrix_bot_postmoogle_smtp_host_bind_port` and `matrix_bot_postmoogle_submission_host_bind_port` variables, but other email servers will try to deliver on these default (standard) ports, so changing them is of little use.


## Adjusting the playbook configuration

Add the following configuration to your `inventory/host_vars/matrix.example.com/vars.yml` file:

```yaml
matrix_bot_postmoogle_enabled: true

# Uncomment and adjust this part if you'd like to use a username different than the default
# matrix_bot_postmoogle_login: postmoogle

# Generate a strong password here. Consider generating it with `pwgen -s 64 1`
matrix_bot_postmoogle_password: PASSWORD_FOR_THE_BOT

# Uncomment to add one or more admins to this bridge:
#
# matrix_bot_postmoogle_admins:
#  - '@yourAdminAccount:{{ matrix_domain }}'
#
# .. unless you've made yourself an admin of all bots/bridges like this:
#
# matrix_admin: '@yourAdminAccount:{{ matrix_domain }}'
```

## Adjusting DNS records

You will also need to add several DNS records so that Postmoogle can send emails. See [Configuring DNS](configuring-dns.md) for details about DNS changes.

## Installing

After configuring the playbook, run the [installation](installing.md) command:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,ensure-matrix-users-created,start
```

**Notes**:

- the `ensure-matrix-users-created` playbook tag makes the playbook automatically create a user account of the bridge's bot

- if you change the bridge's bot password (`matrix_bot_postmoogle_password` in your `vars.yml` file) subsequently, the bot user's credentials on the homeserver won't be updated automatically. If you'd like to change the bot user's password, use a tool like [synapse-admin](configuring-playbook-synapse-admin.md) to change it, and then update `matrix_bot_postmoogle_password` to let the bot know its new password


## Usage

To use the bridge, invite the `@postmoogle:example.com` bot user into a room you want to use as a mailbox.

Then send `!pm mailbox NAME` to expose this Matrix room as an inbox with the email address `NAME@matrix.example.com`. Emails sent to that email address will be forwarded to the room.

Send `!pm help` to the room to see the bridge's help menu for additional commands.

You can also refer to the upstream [documentation](https://github.com/etkecc/postmoogle).

### Debug/Logs

As with all other services, you can find their logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by running something like `journalctl -fu matrix-bot-postmoogle`

The default logging level for this bridge is `INFO`, but you can increase it to `DEBUG` with the following additional configuration:

```yaml
matrix_bot_postmoogle_loglevel: 'DEBUG'
```
