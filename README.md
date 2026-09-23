"""
ITECC04 Laboratory 4
Complete Code: Stack, Expression, Circular Queue, and Deque
"""


"""ITECC04 Laboratory 4, Part D: the circular queue and the deque."""

class CircularQueue:

    def __init__(self, capacity):
        """Step 1."""
        if capacity < 1:
            raise ValueError("capacity must be at least 1")

        self._items = [None] * capacity
        self._front = 0
        self._count = 0
        self._capacity = capacity

    def enqueue(self, item):
        """Step 2."""
        if self.is_full():
            raise OverflowError("queue is full")

        rear = (self._front + self._count) % self._capacity
        self._items[rear] = item
        self._count += 1

    def dequeue(self):
        """Step 3."""
        if self.is_empty():
            raise IndexError("queue is empty")

        item = self._items[self._front]
        self._items[self._front] = None
        self._front = (self._front + 1) % self._capacity
        self._count -= 1

        return item

    def peek(self):
        """Step 4."""
        if self.is_empty():
            raise IndexError("queue is empty")

        return self._items[self._front]

    def is_empty(self):
        """Step 5."""
        return self._count == 0

    def is_full(self):
        """Step 6."""
        return self._count == self._capacity

    def size(self):
        """Step 7."""
        return self._count

    def slots(self):
        """Written for you."""
        return list(self._items)


class Deque:
    """A queue you may add to and remove from at both ends."""

    def __init__(self):
        """Step 8."""
        self._items = []

    def add_front(self, item):
        """Step 9."""
        self._items.insert(0, item)

    def add_rear(self, item):
        """Step 10."""
        self._items.append(item)

    def remove_front(self):
        """Step 11."""
        if self.is_empty():
            raise IndexError("deque is empty")

        return self._items.pop(0)

    def remove_rear(self):
        """Step 12."""
        if self.is_empty():
            raise IndexError("deque is empty")

        return self._items.pop()

    def is_empty(self):
        """Step 13."""
        return len(self._items) == 0

    def size(self):
        """Step 14."""
        return len(self._items)


def is_palindrome(text):
    """Step 15."""
    deque = Deque()

    # Load only letters and ignore case
    for char in text:
        if char.isalpha():
            deque.add_rear(char.lower())

    # Compare front and rear
    while deque.size() > 1:
        front = deque.remove_front()
        rear = deque.remove_rear()

        if front != rear:
            return False

    return True


"""ITECC04 Laboratory 4, Parts B and C: the converter and the evaluator.

Part B turns infix into postfix using the Shunting Yard algorithm.
Part C evaluates a postfix expression.

Both use your own stack. Import it, do not use a bare Python list.

TOKENS ARE SEPARATED BY SPACES.
"""


# Higher number binds tighter.
PRECEDENCE = {
    "+": 1,
    "-": 1,
    "*": 2,
    "/": 2,
    "%": 2,
    "^": 3
}

# ^ is right-associative.
RIGHT_ASSOCIATIVE = {"^"}


def tokenize(expression):
    """Splits expression on whitespace."""
    return expression.split()


def infix_to_postfix(expression):
    """Step 1. Convert infix expression into postfix."""

    output = []
    operators = ArrayStack()

    for token in tokenize(expression):

        # -------------------------
        # OPERAND
        # -------------------------
        if token not in PRECEDENCE and token not in ("(", ")"):
            output.append(token)

        # -------------------------
        # LEFT PARENTHESIS
        # -------------------------
        elif token == "(":
            operators.push(token)

        # -------------------------
        # RIGHT PARENTHESIS
        # -------------------------
        elif token == ")":

            found_left = False

            while not operators.is_empty():

                top = operators.pop()

                if top == "(":
                    found_left = True
                    break

                output.append(top)

            if not found_left:
                raise ValueError("unbalanced parentheses")

        # -------------------------
        # OPERATOR
        # -------------------------
        else:

            while not operators.is_empty():

                top = operators.peek()

                # Left parenthesis acts as a fence.
                if top == "(":
                    break

                # Pop higher precedence operators.
                if PRECEDENCE[top] > PRECEDENCE[token]:
                    output.append(operators.pop())

                # Pop equal precedence unless current operator
                # is right associative.
                elif (
                    PRECEDENCE[top] == PRECEDENCE[token]
                    and token not in RIGHT_ASSOCIATIVE
                ):
                    output.append(operators.pop())

                else:
                    break

            operators.push(token)

    # -------------------------
    # DRAIN OPERATOR STACK
    # -------------------------
    while not operators.is_empty():

        top = operators.pop()

        if top == "(":
            raise ValueError("unbalanced parentheses")

        output.append(top)

    return " ".join(output)


def evaluate_postfix(expression):
    """Step 2. Evaluate postfix expression. Return a float."""

    values = ArrayStack()

    for token in tokenize(expression):

        # -------------------------
        # OPERATOR
        # -------------------------
        if token in PRECEDENCE:

            # Need at least two operands.
            if values.is_empty():
                raise ValueError("not enough operands")

            # FIRST pop = RIGHT operand
            right = values.pop()

            # SECOND pop = LEFT operand
            if values.is_empty():
                raise ValueError("not enough operands")

            left = values.pop()

            # Calculate result.
            result = apply_operator(token, left, right)

            # Put result back onto stack.
            values.push(result)

        # -------------------------
        # OPERAND
        # -------------------------
        else:

            try:
                number = float(token)
                values.push(number)

            except ValueError:
                raise ValueError("invalid operand")

    # No values means invalid expression.
    if values.is_empty():
        raise ValueError("invalid expression")

    # There should be exactly ONE value.
    result = values.pop()

    # More than one value = malformed expression.
    if not values.is_empty():
        raise ValueError("too many operands")

    return result


def apply_operator(operator, left, right):
    """Step 3. Apply an operator to left and right."""

    # -------------------------
    # ADDITION
    # -------------------------
    if operator == "+":
        return left + right

    # -------------------------
    # SUBTRACTION
    # -------------------------
    elif operator == "-":
        return left - right

    # -------------------------
    # MULTIPLICATION
    # -------------------------
    elif operator == "*":
        return left * right

    # -------------------------
    # DIVISION
    # -------------------------
    elif operator == "/":

        if right == 0:
            raise ZeroDivisionError("division by zero")

        return left / right

    # -------------------------
    # MODULO
    # -------------------------
    elif operator == "%":

        if right == 0:
            raise ZeroDivisionError("modulo by zero")

        return left % right

    # -------------------------
    # POWER
    # -------------------------
    elif operator == "^":
        return left ** right

    # -------------------------
    # UNKNOWN OPERATOR
    # -------------------------
    else:
        raise ValueError("unknown operator")


def convert_and_evaluate(expression):
    """Convert infix to postfix, then evaluate it."""

    postfix = infix_to_postfix(expression)

    value = evaluate_postfix(postfix)

    return postfix, value


if _name_ == "_main_":

    try:
        postfix, value = convert_and_evaluate("3 + 4 * 2")

        print("infix   : 3 + 4 * 2")
        print("postfix :", postfix)
        print("value   :", value)

    except NotImplementedError as unfinished:

        print("Not written yet ->", unfinished)


"""ITECC04 Laboratory 4, Part A2: the linked-list-based stack.

Same public interface as ArrayStack. Different storage, same contract.
"""


class Node:
    """Written for you."""

    _slots_ = ("value", "next")

    def __init__(self, value, nxt=None):
        self.value = value
        self.next = nxt


class LinkedStack:

    def __init__(self):
        """Step 1. Create an empty stack."""
        self._top = None
        self._size = 0

    def push(self, item):
        """Step 2. Add an item to the top."""
        new_node = Node(item, self._top)
        self._top = new_node
        self._size += 1

    def pop(self):
        """Step 3. Remove and return the top item."""
        if self.is_empty():
            raise IndexError("pop from empty stack")

        node = self._top
        self._top = node.next
        self._size -= 1

        return node.value

    def peek(self):
        """Step 4. Return the top item without removing it."""
        if self.is_empty():
            raise IndexError("peek from empty stack")

        return self._top.value

    def is_empty(self):
        """Step 5. Check whether the stack is empty."""
        return self._top is None

    def size(self):
        """Step 6. Return the number of items."""
        return self._size

    def __len__(self):
        """Written for you."""
        return self.size()


"""ITECC04 Laboratory 4, Part A1: the array-based stack."""

class ArrayStack:

    def __init__(self):
        """Step 1. Create the empty list that will hold the items."""
        self._items = []

    def push(self, item):
        """Step 2. Put an item on top."""
        self._items.append(item)

    def pop(self):
        """Step 3. Remove and return the top item."""
        if self.is_empty():
            raise IndexError("pop from an empty stack")

        return self._items.pop()

    def peek(self):
        """Step 4. Return the top item without removing it."""
        if self.is_empty():
            raise IndexError("peek from an empty stack")

        return self._items[-1]

    def is_empty(self):
        """Step 5. Return True when the stack is empty."""
        return len(self._items) == 0

    def size(self):
        """Step 6. Return the number of items."""
        return len(self._items)

    def __len__(self):
        """Allows len(stack)."""
        return self.size()
