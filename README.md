# Reto-7
The restaurant class revisted like for the third time.
Add the proper data structure to manage multiple orders (maybe a FIFO queue)
Define a named tuple somewhere in the menu, e.g. to define a set of items.
Create an interface in the order class, to create a new menu, aggregate the functions for add, update, delete items. All the menus should be stored as JSON files. (use dicts for this task.)

**Descrpción del ejercicio**

Este código implementa un sistema completo para la gestión de un restaurante. Define diferentes tipos de productos del menú —bebidas, entradas y platos fuertes— mediante clases con atributos específicos y métodos para calcular precios, incluyendo ajustes como descuentos o variaciones por tamaño. También incluye un `MenuManager`, encargado de agregar, modificar, eliminar y almacenar los ítems del menú en archivos JSON, permitiendo guardar y cargar la información de manera persistente. Además, incorpora la clase `Order`, que representa un pedido conformado por múltiples ítems y permite calcular su costo total aplicando descuentos según ciertas reglas. Finalmente, el sistema maneja múltiples pedidos mediante una estructura FIFO (`OrderQueue`) basada en una cola, asegurando que los pedidos se procesen en el orden en que fueron recibidos.
```python
import json
from typing import List, Optional
from collections import deque

CATEGORIES = ("bebidas", "entradas", "platos_fuertes")


class MenuItem:
    def __init__(self, name: str, price: float):
        self.__name = str(name)
        self.__price = float(price)

    def get_name(self) -> str:
        return self.__name

    def set_name(self, name: str):
        self.__name = str(name)

    def get_price(self) -> float:
        return self.__price

    def set_price(self, price: float):
        self.__price = float(price)

    def get_total_price(self, order=None) -> float:
        return self.__price

    def to_dict(self):
        """Convierte el ítem a un diccionario apto para JSON."""
        return {
            "type": self.__class__.__name__,
            "name": self.__name,
            "price": self.__price
        }


class Beverage(MenuItem):
    def __init__(self, name: str, price: float, size: str):
        super().__init__(name, price)
        self.__size = size.lower()  # pequeño, mediano, grande

    def get_size(self) -> str:
        return self.__size

    def set_size(self, size: str):
        self.__size = size.lower()

    def get_total_price(self, order=None) -> float:
        base = self.get_price()
        size_factor = {"pequeño": 0.8, "mediano": 1.0, "grande": 1.2}
        price = base * size_factor.get(self.__size, 1.0)

        if order is not None and order.has_main_course():
            price *= 0.8  # 20% descuento

        return price

    def to_dict(self):
        data = super().to_dict()
        data["size"] = self.__size
        return data


class Appetizer(MenuItem):
    def __init__(self, name: str, price: float, is_vegan: bool):
        super().__init__(name, price)
        self.__is_vegan = bool(is_vegan)

    def to_dict(self):
        data = super().to_dict()
        data["is_vegan"] = self.__is_vegan
        return data


class MainCourse(MenuItem):
    def __init__(self, name: str, price: float, calories: int):
        super().__init__(name, price)
        self.__calories = calories

    def to_dict(self):
        data = super().to_dict()
        data["calories"] = self.__calories
        return data


class MenuManager:
    """
    Maneja un menú completo con:
    - agregar ítems
    - actualizar ítems
    - eliminar ítems
    - guardar en JSON
    - cargar desde JSON
    """

    def __init__(self):
        self.menu = {
            "bebidas": [],
            "entradas": [],
            "platos_fuertes": []
        }

    def add_item(self, category: str, item: MenuItem):
        if category not in self.menu:
            raise ValueError("Categoría inválida")
        self.menu[category].append(item.to_dict())

    def update_item(self, category: str, index: int, new_item: MenuItem):
        if category not in self.menu:
            raise ValueError("Categoría inválida")
        if not (0 <= index < len(self.menu[category])):
            raise IndexError("Índice fuera de rango")
        self.menu[category][index] = new_item.to_dict()

    def delete_item(self, category: str, index: int):
        if category not in self.menu:
            raise ValueError("Categoría inválida")
        if not (0 <= index < len(self.menu[category])):
            raise IndexError("Índice fuera de rango")
        del self.menu[category][index]

    def save_to_json(self, filename="menu.json"):
        with open(filename, "w", encoding="utf-8") as f:
            json.dump(self.menu, f, indent=4, ensure_ascii=False)

    def load_from_json(self, filename="menu.json"):
        with open(filename, "r", encoding="utf-8") as f:
            self.menu = json.load(f)


class Order:
    def __init__(self):
        self.__items: List[MenuItem] = []

    def add_item(self, item: MenuItem):
        self.__items.append(item)

    def get_items(self):
        return list(self.__items)

    def has_main_course(self):
        return any(isinstance(i, MainCourse) for i in self.__items)

    def calculate_total(self):
        return sum(item.get_total_price(self) for item in self.__items)

    def apply_discount(self):
        t = self.calculate_total()
        if len(self.__items) >= 5:
            return t * 0.9
        return t

    def to_dict(self):
        return [item.to_dict() for item in self.__items]
    
    
class OrderQueue:
    """Cola FIFO para manejar múltiples pedidos."""
    def __init__(self):
        self.queue = deque()

    def add_order(self, order: Order):
        self.queue.append(order)

    def next_order(self) -> Optional[Order]:
        if self.queue:
            return self.queue.popleft()
        return None  # cola vacía

    def __len__(self):
        return len(self.queue)


if __name__ == "__main__":
    # Crear gestor de menú
    manager = MenuManager()

    # Crear ítems
    bebida = Beverage("Limonada", 5000, "grande")
    entrada = Appetizer("Patacones", 7000, True)
    plato = MainCourse("Bandeja paisa", 20000, 1200)

    # Agregar ítems al menú
    manager.add_item("bebidas", bebida)
    manager.add_item("entradas", entrada)
    manager.add_item("platos_fuertes", plato)

    # Guardar JSON
    manager.save_to_json()

    print("Menú guardado en JSON.")

    # Crear pedidos
    order1 = Order()
    order1.add_item(bebida)
    order1.add_item(plato)

    order2 = Order()
    order2.add_item(entrada)

    # Manejo con cola FIFO
    queue = OrderQueue()
    queue.add_order(order1)
    queue.add_order(order2)

    print("Pedidos en cola:", len(queue))

    siguiente = queue.next_order()
    print("Procesando pedido:", siguiente.to_dict())
```
 
