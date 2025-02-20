��#   D A O 
 # CONEXION.PY
from mysql.connector import pooling
from mysql.connector import Error

class Conexion:
    DATABASE = 'zona_fit_db'
    USERNAME = 'root'
    PASSWORD = 'Leomessi2487_'
    DB_PORT = '3306'
    HOST = 'localhost'
    POOL_SIZE = 5
    POOL_NAME = 'zona_fit_pool'
    pool = None

    @classmethod
    def obtener_pool(cls):
        if cls.pool is None: # Se crea un objeto pool
            try:
                cls.pool = pooling.MySQLConnectionPool(
                    pool_name = cls.POOL_NAME,
                    pool_size = cls.POOL_SIZE,
                    host = cls.HOST,
                    port = cls.DB_PORT,
                    database = cls.DATABASE,
                    user = cls.USERNAME,
                    password = cls.PASSWORD
                )
                #print(f'Nombre del pool: {cls.POOL_NAME}')
                #print(f'Tamaño del pool: {cls.POOL_SIZE}')
                return cls.pool
            except Error as e:
                print(f'Ocurrió un error al obtener el pool {e}')
        else:
            return cls.pool     
        
    @classmethod
    def obtener_conexion(cls):
        return cls.obtener_pool().get_connection()

    @classmethod
    def liberar_conexion(cls, conexion):
        conexion.close()

if __name__ == '__main__':
    # Creacion del objeto pool
    pool = Conexion.obtener_pool()
    print(pool)           

    # Obtener un objeto conexion
    cnx1 = Conexion.obtener_conexion()
    print(cnx1)
    Conexion.liberar_conexion(cnx1)


# CLIENTE.PY

class Cliente:
    def __init_(self, id=None, nombre=None, apellido=None, membresia=None):
        self.id = id
        self.nombre = nombre
        self.apellido = apellido
        self.membresia = membresia

    def __str__(self):
        return f'Id: {self.id}, Nombre: {self.nombre}, Apellido: {self.apellido}, Membresia: {self.membresia}'    








# CLIENTE_DAO.PY

from cliente import Cliente
from conexion import Conexion

class ClienteDAO:
    SELECCIONAR = 'SELECT id, nombre, apellido, membresia FROM cliente'
    INSERTAR = 'INSERT INTO cliente(nombre, apellido, membresia) VALUES(%s, %s, %s)'
    ACTUALIZAR = 'UPDATE cliente SET nombre = %s, apellido = %s, membresia = %s WHERE id = %s'
    ELIMINAR = 'DELETE FROM cliente WHERE id = %s'

    @classmethod
    def seleccionar(cls):
        conexion = None
        cursor = None
        try:
            conexion = Conexion.obtener_conexion()
            cursor = conexion.cursor()
            cursor.execute(cls.SELECCIONAR)
            registros = cursor.fetchall()
            print(f"Registros obtenidos: {registros}")  # Depuración
            clientes = []
            for registro in registros:
                cliente_obj = Cliente(*registro)  # Desempaquetar los registros
                clientes.append(cliente_obj)
            return clientes
        except Exception as e:
            print(f'Ocurrió un error al seleccionar clientes: {e}')
        finally:
            if cursor:
                cursor.close()
            if conexion:
                Conexion.liberar_conexion(conexion)

if __name__ == '__main__':
    clientes = ClienteDAO.seleccionar() or []  # Asegurar que no sea None
    if not clientes:
        print("No hay clientes disponibles.")
    for cliente in clientes:
        print(cliente)
        

    
