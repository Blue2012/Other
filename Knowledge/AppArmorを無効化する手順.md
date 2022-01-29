以下のコマンドを実行してAppArmorの設定ファイルを開きます。

```bash
sudo vi /etc/default/grub
```

出力された文字列内の「GRUB_CMDLINE_LINUX」の値を以下に変更し、保存します。

```vim
GRUB_CMDLINE_LINUX="apparmor=0"
```

編集後、以下のコマンドを実行してGrubの設定を更新します。

```bash
sudo update-grub
```

OSを再起動します。

```bash
sudo reboot
```

なお、AppArmor の無効化の確認は以下のコマンドでおこなえます。

```bash
sudo apparmor_status
```

無効化の手順を実施した場合、上記のコマンドを実行すると、以下の結果が表示されます。
```bash
apparmor module is loaded.
apparmor filesystem is not mounted.
```
