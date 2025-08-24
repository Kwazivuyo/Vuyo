#########################################################################
#                                   
#   Swiss crypto tool is created purely in python. Supports
#   different hashing algorithms and encryption algorithms.
#   Users can send secure emails by encrypting their body.
#   Compatible with Python 3.x
#   Author : Kwazi Vuyo
#   Email  : vuyom@takenoteit.co.za
#########################################################################

import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from Crypto.Cipher import ARC4, DES, AES
import tkinter as tk
from tkinter import messagebox
import base64
import hashlib
import os
import binascii

root = tk.Tk()
root.title("Secure Mail")

text_var = tk.StringVar()
var = tk.StringVar()

result_label = tk.Label(root, text="", font=("Arial", 14), fg="blue")
result_label.pack()

entry = tk.Entry(root, width=50)
entry.pack()

# ---------------- Hashing Functions ---------------- #
def md5():
    data = entry.get().encode()
    result = hashlib.md5(data).hexdigest()
    result_label.config(text="MD5 = " + result)
    print("MD5:", result)

def sha1():
    data = entry.get().encode()
    result = hashlib.sha1(data).hexdigest()
    result_label.config(text="SHA1 = " + result)
    print("SHA1:", result)

def sha224():
    data = entry.get().encode()
    result = hashlib.sha224(data).hexdigest()
    result_label.config(text="SHA224 = " + result)
    print("SHA224:", result)

def sha256():
    data = entry.get().encode()
    result = hashlib.sha256(data).hexdigest()
    result_label.config(text="SHA256 = " + result)
    print("SHA256:", result)

def sha384():
    data = entry.get().encode()
    result = hashlib.sha384(data).hexdigest()
    result_label.config(text="SHA384 = " + result)
    print("SHA384:", result)

def sha512():
    data = entry.get().encode()
    result = hashlib.sha512(data).hexdigest()
    result_label.config(text="SHA512 = " + result)
    print("SHA512:", result)

# ---------------- DES Functions ---------------- #
def des_encrypt():
    data = entry.get().encode()
    key = b"12345678"
    des = DES.new(key, DES.MODE_ECB)
    padded = data + b' ' * (8 - len(data) % 8)
    cipher_text = des.encrypt(padded)
    hex_text = binascii.hexlify(cipher_text).decode()
    result_label.config(text="DES Encrypted = " + hex_text)
    print("DES Encrypted:", hex_text)

def des_decrypt():
    hex_text = entry.get()
    key = b"12345678"
    des = DES.new(key, DES.MODE_ECB)
    cipher_bytes = binascii.unhexlify(hex_text)
    decrypted = des.decrypt(cipher_bytes).rstrip(b' ').decode()
    result_label.config(text="DES Decrypted = " + decrypted)
    print("DES Decrypted:", decrypted)

# ---------------- AES Functions ---------------- #
def aes_encrypt():
    data = entry.get().encode()
    block_size = 32
    padding = b'{'
    pad = lambda s: s + (block_size - len(s) % block_size) * padding
    key = os.urandom(block_size)
    cipher = AES.new(key)
    encrypted = cipher.encrypt(pad(data))
    hex_text = binascii.hexlify(encrypted).decode()
    key_hex = binascii.hexlify(key).decode()
    result_label.config(text="AES Encrypted = " + hex_text)
    print("AES Key:", key_hex)
    print("AES Encrypted:", hex_text)

def aes_decrypt():
    key_hex = entry.get()  # First entry should contain key
    cipher_hex = entry.get()  # Replace with actual encrypted text entry
    key = binascii.unhexlify(key_hex)
    cipher_bytes = binascii.unhexlify(cipher_hex)
    block_size = 32
    padding = b'{'
    decode = lambda c, e: c.decrypt(e).rstrip(padding)
    cipher = AES.new(key)
    decrypted = decode(cipher, cipher_bytes).decode()
    result_label.config(text="AES Decrypted = " + decrypted)
    print("AES Decrypted:", decrypted)

# ---------------- ARC4 Functions ---------------- #
def arc4_encrypt():
    key = entry.get().encode()
    data = entry.get().encode()
    cipher = ARC4.new(key)
    encrypted = cipher.encrypt(data)
    hex_text = binascii.hexlify(encrypted).decode()
    result_label.config(text="ARC4 Encrypted = " + hex_text)
    print("ARC4 Encrypted:", hex_text)

def arc4_decrypt():
    key = entry.get().encode()
    cipher_hex = entry.get()
    data = binascii.unhexlify(cipher_hex)
    cipher = ARC4.new(key)
    decrypted = cipher.decrypt(data).decode()
    result_label.config(text="ARC4 Decrypted = " + decrypted)
    print("ARC4 Decrypted:", decrypted)

# ---------------- Secure Email ---------------- #
def send_email():
    Semail = entry.get()
    Temail = entry.get()
    body_text = entry.get()
    user_name = entry.get()
    password = entry.get()

    msg = MIMEMultipart()
    msg['From'] = Semail
    msg['To'] = Temail
    msg['Subject'] = "Encrypted Email"

    # AES encrypt body
    block_size = 32
    padding = b'{'
    pad = lambda s: s + (block_size - len(s) % block_size) * padding
    key = os.urandom(block_size)
    cipher = AES.new(key)
    encrypted_body = binascii.hexlify(cipher.encrypt(pad(body_text.encode()))).decode()
    msg.attach(MIMEText(encrypted_body))

    # Send email
    server = smtplib.SMTP('smtp.gmail.com:587')
    server.starttls()
    server.login(user_name, password)
    server.sendmail(Semail, Temail, msg.as_string())
    server.quit()
    messagebox.showinfo("Success", "Email sent successfully!")

# ---------------- GUI Menu ---------------- #
menubar = tk.Menu(root)
hash_menu = tk.Menu(menubar, tearoff=0)
hash_menu.add_command(label="MD5", command=md5)
hash_menu.add_command(label="SHA1", command=sha1)
hash_menu.add_command(label="SHA224", command=sha224)
hash_menu.add_command(label="SHA256", command=sha256)
hash_menu.add_command(label="SHA384", command=sha384)
hash_menu.add_command(label="SHA512", command=sha512)
menubar.add_cascade(label="Hash Converter", menu=hash_menu)

encrypt_menu = tk.Menu(menubar, tearoff=0)
encrypt_menu.add_command(label="DES Encrypt", command=des_encrypt)
encrypt_menu.add_command(label="AES Encrypt", command=aes_encrypt)
encrypt_menu.add_command(label="ARC4 Encrypt", command=arc4_encrypt)
menubar.add_cascade(label="Encryption", menu=encrypt_menu)

decrypt_menu = tk.Menu(menubar, tearoff=0)
decrypt_menu.add_command(label="DES Decrypt", command=des_decrypt)
decrypt_menu.add_command(label="AES Decrypt", command=aes_decrypt)
decrypt_menu.add_command(label="ARC4 Decrypt", command=arc4_decrypt)
menubar.add_cascade(label="Decryption", menu=decrypt_menu)

email_menu = tk.Menu(menubar, tearoff=0)
email_menu.add_command(label="Send Secure Email", command=send_email)
menubar.add_cascade(label="Secure Email", menu=email_menu)

root.config(menu=menubar)
root.mainloop()
