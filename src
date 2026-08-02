import argparse
import socket
from concurrent.futures import ThreadPoolExecutor, as_completed

SERVICE_NAMES = {
    20: 'FTP (data)',
    21: 'FTP (control)',
    22: 'SSH',
    23: 'Telnet',
    25: 'SMTP',
    53: 'DNS',
    67: 'DHCP',
    68: 'DHCP',
    80: 'HTTP',
    110: 'POP3',
    123: 'NTP',
    135: 'RPC',
    139: 'NetBIOS',
    143: 'IMAP',
    161: 'SNMP',
    194: 'IRC',
    220: 'IMAP3',
    389: 'LDAP',
    443: 'HTTPS',
    445: 'SMB',
    465: 'SMTPS',
    514: 'Syslog',
    587: 'SMTP (submission)',
    631: 'IPP',
    636: 'LDAPS',
    873: 'rsync',
    993: 'IMAPS',
    995: 'POP3S',
    1080: 'SOCKS',
    1433: 'MSSQL',
    1521: 'Oracle',
    1723: 'PPTP',
    2049: 'NFS',
    3306: 'MySQL',
    3389: 'RDP',
    5432: 'PostgreSQL',
    5900: 'VNC',
    6379: 'Redis',
    8080: 'HTTP-alt',
    8443: 'HTTPS-alt',
}


def get_service_name(port: int) -> str:
    return SERVICE_NAMES.get(port, 'desconhecido')


def decode_banner(data: bytes) -> str:
    text = data.decode('utf-8', errors='ignore').strip()
    return ' '.join(text.split()) if text else ''


def grab_banner(sock: socket.socket, host: str, port: int, timeout: float = 1.0) -> str:
    sock.settimeout(timeout)
    try:
        data = sock.recv(1024)
    except socket.timeout:
        data = b''
    except OSError:
        return ''

    if data:
        return decode_banner(data)

    if port in {80, 8000, 8080, 3000, 5000, 8888}:
        try:
            request = f'GET / HTTP/1.0\r\nHost: {host}\r\n\r\n'
            sock.sendall(request.encode('utf-8', errors='ignore'))
            data = sock.recv(1024)
            return decode_banner(data)
        except (socket.timeout, OSError):
            return ''

    return ''


def scan_port(target: str, port: int, timeout: float = 1.0) -> tuple[bool, str]:
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.settimeout(timeout)
            if s.connect_ex((target, port)) != 0:
                return False, ''
            banner = grab_banner(s, target, port, timeout)
            return True, banner
    except socket.gaierror:
        raise ValueError(f'Alvo inválido: {target}')
    except OSError as e:
        raise ConnectionError(f'Erro de conexão ao testar {target}:{port}: {e}')


def parse_ports(ports: str) -> list[int]:
    if '-' in ports:
        start, end = ports.split('-', 1)
        start_port = int(start)
        end_port = int(end)
        if start_port < 1 or end_port > 65535 or start_port > end_port:
            raise ValueError('Intervalo de portas inválido. Use algo como 20-1024.')
        return list(range(start_port, end_port + 1))
    else:
        port = int(ports)
        if port < 1 or port > 65535:
            raise ValueError('Porta inválida. Use um número entre 1 e 65535.')
        return [port]


def scan_ports_concurrently(target: str, ports: list[int], timeout: float = 1.0, max_workers: int = 50) -> list[int]:
    open_ports: list[int] = []
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = {executor.submit(scan_port, target, port, timeout): port for port in ports}
        for future in as_completed(futures):
            port = futures[future]
            try:
                opened, banner = future.result()
                if opened:
                    service = get_service_name(port)
                    banner_text = f' - banner: {banner}' if banner else ''
                    print(f'porta {port} está aberta ({service}){banner_text}')
                    open_ports.append(port)
                else:
                    print(f'porta {port} está fechada')
            except ValueError as err:
                print(f'Erro fatal: {err}')
                return []
            except ConnectionError as err:
                print(f'Aviso: porta {port} não pôde ser testada ({err})')
                continue
    return sorted(open_ports)


def main() -> None:
    parser = argparse.ArgumentParser(description='Scanner simples de portas TCP.')
    parser.add_argument('target', help='Host ou IP de destino (ex: 127.0.0.1 ou hydra.local)')
    parser.add_argument('-p', '--ports', default='22', help='Porta única ou intervalo (ex: 22 ou 20-1024)')
    parser.add_argument('-t', '--threads', type=int, default=50, help='Número de threads para scan paralelo')
    parser.add_argument('--timeout', type=float, default=1.0, help='Tempo limite de conexão em segundos')
    args = parser.parse_args()

    try:
        ports = parse_ports(args.ports)
    except ValueError as err:
        print(f'Erro: {err}')
        return

    if len(ports) == 1:
        print(f'Testando {args.target} na porta {ports[0]}')
    else:
        print(f'Testando {args.target} nas portas: {ports[0]}-{ports[-1]} com {args.threads} threads')

    try:
        open_ports = scan_ports_concurrently(args.target, ports, timeout=args.timeout, max_workers=args.threads)
    except ValueError as err:
        print(f'Erro: {err}')
        return
    except ConnectionError as err:
        print(f'Erro: {err}')
        return

    if open_ports:
        print(f'Portas abertas encontradas: {open_ports}')
    else:
        print('Nenhuma porta aberta encontrada.')


if __name__ == '__main__':
    main()
