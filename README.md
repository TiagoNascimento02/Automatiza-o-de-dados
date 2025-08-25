import os
from ftplib import FTP
import paramiko
from datetime import datetime

# 📁 Pasta de destino
pasta_local = "dados_recebidos"
os.makedirs(pasta_local, exist_ok=True)

# 📅 Mês atual em caixa alta (ex: "AGOSTO")
mes_atual = datetime.now().strftime('%B').upper()

# 🌐 Host padrão (para FTP)
ftp_host = "ftp-svr.iscam.com"
sftp_port = 22  # Porta padrão para SFTP

# 👥 Empresas com seus dados
empresas = [
   {"codigo": "usuario", "usuario": "usuario", "senha": "senha", "tipo": "FTP"},
# ⚙️ Função para baixar via FTP
def baixar_ftp(codigo, usuario, senha):
    try:
        print(f'📡 FTP - Empresa {codigo}')
        ftp = FTP(ftp_host)
        ftp.login(user=usuario, passwd=senha)
        arquivos = ftp.nlst()
        for nome in arquivos:
            if mes_atual in nome.upper() and nome.endswith(".xls"):
                caminho_local = os.path.join(pasta_local, f'{codigo}_{nome}')
                with open(caminho_local, 'wb') as f:
                    ftp.retrbinary(f'RETR {nome}', f.write)
                print(f'✅ Baixado via FTP: {caminho_local}')
        ftp.quit()
    except Exception as e:
        print(f'❌ Erro FTP {codigo}: {e}')

# 🔐 Função para baixar via SFTP
def baixar_sftp(codigo, usuario, senha):
    try:
        print(f'🔐 SFTP - Empresa {codigo}')
        transporte = paramiko.Transport((ftp_host, sftp_port))
        transporte.connect(username=usuario, password=senha)
        sftp = paramiko.SFTPClient.from_transport(transporte)
        arquivos = sftp.listdir()
        for nome in arquivos:
            if mes_atual in nome.upper() and nome.endswith(".xls"):
                caminho_local = os.path.join(pasta_local, f'{codigo}_{nome}')
                sftp.get(nome, caminho_local)
                print(f'✅ Baixado via SFTP: {caminho_local}')
        sftp.close()
        transporte.close()
    except Exception as e:
        print(f'❌ Erro SFTP {codigo}: {e}')

# 🚀 Executa o download para cada empresa
for emp in empresas:
    if emp["tipo"] == "FTP":
        baixar_ftp(emp["codigo"], emp["usuario"], emp["senha"])
    elif emp["tipo"] == "SFTP":
        baixar_sftp(emp["codigo"], emp["usuario"], emp["senha"])
    else:
        print(f'⚠️ Tipo desconhecido: {emp["codigo"]}')
