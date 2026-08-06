# Choppus


import sys
import re
from datetime import datetime

class ChoppusInterpreter:
    def __init__(self):
        # Underlying 30,000 cell memory array (Brainfuck/Assembly core)
        self.memory = [0] * 30000
        self.pointer = 0
        # Environment storage for Chef-style ingredients
        self.ingredients = {}

    def run(self, filepath):
        with open(filepath, 'r', encoding='utf-8') as f:
            lines = f.readlines()

        # 1. System Clock Constraint Check (Semicolon Verification)
        total_semicolons = sum(line.count(';') for line in lines)
        current_day = datetime.now().day

        if total_semicolons != current_day:
            print("CRITICAL COMPILATION ERROR: System Clock Constraint Violation.")
            print(f"-> Local Machine Date: Day {current_day}")
            print(f"-> Codebase Semicolon Count: {total_semicolons}")
            print("Execution halted. Semicolon matrix must match the calendar day.")
            sys.exit(1)

        for line_num, line in enumerate(lines, 1):
            raw_line = line.rstrip('\r\n')
            
            # Whitespace Pointer Manipulation (Trailing spaces shift the array pointer)
            trailing_whitespaces = len(raw_line) - len(raw_line.rstrip(' '))
            if trailing_whitespaces > 0:
                self.pointer = (self.pointer + trailing_whitespaces) % 30000

            stripped = line.strip()
            if not stripped:
                continue

            # Explicit Scope Enclosure Check (Lisp encapsulation)
            if not (stripped.startswith("((") and stripped.endswith("))")):
                print(f"SYNTAX ERROR [Line {line_num}]: Invalid scope encapsulation.")
                print("Enclosure Error: Every statement must be strictly wrapped inside '((' and '))'.")
                sys.exit(1)

            # Isolate token content
            content = stripped[2:-2].strip()

            # Helper function to extract hidden U+200B addresses
            def parse_zero_width_address(token):
                u200b_count = token.count('\u200b')
                clean_token = token.replace('\u200b', '')
                return clean_token, u200b_count

            # 2. Operational Parsing: ZUTAT; (Variable Allocation)
            if content.startswith("ZUTAT; "):
                match = re.search(r"ZUTAT;\s+(\d+)g_([^\s]+);", content)
                if match:
                    value = int(match.group(1))
                    raw_name = match.group(2) + ";"
                    
                    name, memory_address = parse_zero_width_address(raw_name)
                    
                    self.ingredients[name] = {"value": value, "address": memory_address}
                    self.memory[memory_address] = value
                else:
                    print(f"BAD TOKEN ERROR [Line {line_num}]: Invalid 'ZUTAT;' structure.")
                    sys.exit(1)

            # 3. Operational Parsing: LABER; (Destructive Read / Output)
            elif content.startswith("LABER; "):
                raw_name = content.replace("LABER; ", "").strip()
                name, target_address = parse_zero_width_address(raw_name)
                
                if name in self.ingredients:
                    allocated_address = self.ingredients[name]["address"]
                    
                    # Zero-Width Addressing Validation (Segmentation Fault Check)
                    if target_address != allocated_address:
                        print(f"SEGMENTATION FAULT [Line {line_num}]: Invalid pointer resolution.")
                        print(f"-> Attempted Address: index {target_address}")
                        sys.exit(1)
                    
                    # Output Execution
                    print(f"[Async-Output]: {self.memory[target_address]}")
                    
                    # Memory Ownership Purge (Rust Borrow Checker Mechanic)
                    del self.ingredients[name]
                else:
                    print(f"MEMORY OWNERSHIP ERROR [Line {line_num}]: Variable '{name}' is unallocated or has already been purged.")
                    sys.exit(1)

            # 4. Operational Parsing: MIX; (Type Concatenation)
            elif content.startswith("MIX; "):
                tokens = content.split()
                if len(tokens) == 6 and tokens[2] == "MIT;" and tokens[4] == "IN;":
                    z1_clean, _ = parse_zero_width_address(tokens[1])
                    z2_clean, _ = parse_zero_width_address(tokens[3])
                    target_clean, target_address = parse_zero_width_address(tokens[5])
                    
                    if z1_clean in self.ingredients and z2_clean in self.ingredients:
                        # JavaScript-style string coercion
                        concatenated_value = str(self.ingredients[z1_clean]["value"]) + str(self.ingredients[z2_clean]["value"])
                        
                        self.ingredients[target_clean] = {"value": int(concatenated_value), "address": target_address}
                        self.memory[target_address] = int(concatenated_value)
                    else:
                        print(f"ENV ERROR [Line {line_num}]: Missing ingredients for 'MIX;' operation.")
                        sys.exit(1)
                else:
                    print(f"SYNTAX ERROR [Line {line_num}]: Malformed 'MIX;' expression.")
                    sys.exit(1)
            
            else:
                print(f"COMPILER ERROR [Line {line_num}]: Unrecognized token sequence or missing keyword semicolon.")
                sys.exit(1)

        print("\n[SUCCESS]: Compilation finished. Environment closed cleanly.")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python choppus.py <file.chpp>")
    else:
        interpreter = ChoppusInterpreter()
        interpreter.run(sys.argv[1])
