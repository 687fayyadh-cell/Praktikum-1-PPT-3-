# Praktikum-1-PPT-3-

import math
class C:
    RESET  = "\033[0m"
    BOLD   = "\033[1m"
    RED    = "\033[91m"
    GRAY   = "\033[90m"
    WHITE  = "\033[97m"
    GREEN  = "\033[92m"
def parse_function(expr: str):
    expr = expr.replace("^", "**")
    allowed = {
        "sin": math.sin, "cos": math.cos, "tan": math.tan,
        "exp": math.exp, "log": math.log, "log10": math.log10,
        "sqrt": math.sqrt, "abs": abs,
        "pi": math.pi, "e": math.e,
    }
    def f(x):
        return eval(expr, {"__builtins__": {}}, {**allowed, "x": x})
    return f
def secant_method(f, x0: float, x1: float,
                  tol: float = 1e-6, max_iter: int = 100):
    table = []
    converged = False

    for n in range(1, max_iter + 1):
        fx0 = f(x0)
        fx1 = f(x1)

        denom = fx1 - fx0
        if abs(denom) < 1e-15:
            raise ValueError(
                f"Iterasi {n}: Pembagi ≈ 0 (f(x₁) ≈ f(x₀)). "
                "Coba tebakan awal yang berbeda."
            )

        x2 = x1 - fx1 * (x1 - x0) / denom
        error = abs((x2 - x1) / x2) * 100 if x2 != 0 else float('inf')

        table.append({
            "n"    : n,
            "x0"   : x0,
            "x1"   : x1,
            "fx0"  : fx0,
            "fx1"  : fx1,
            "x2"   : x2,
            "error": error,
        })

        x0, x1 = x1, x2

        if error < tol:
            converged = True
            break

    return x1, converged, table

def print_table(table, converged):
    col = [6, 14, 14, 14, 14, 14, 14]
    headers = ["  n", "    x₀", "    x₁", "    f(x₀)", "    f(x₁)", "    x₂", "    Er (%)"]

    sep = C.GRAY + "+" + "+".join("─" * (w + 2) for w in col) + "+" + C.RESET
    header_line = C.GRAY + "|" + C.RESET
    for h, w in zip(headers, col):
        header_line += C.BOLD + C.BOLD + f" {h:<{w}} " + C.RESET + C.GRAY + "|" + C.RESET

    print(sep)
    print(header_line)
    print(sep)

    for i, row in enumerate(table):
        is_last = (i == len(table) - 1) and converged
        color = C.GREEN if is_last else C.WHITE

        def fmtv(v):
            return f"{v:.5f}"

        line = C.GRAY + "|" + C.RESET
        vals = [
            f"{row['n']:>4}",
            fmtv(row['x0']),
            fmtv(row['x1']),
            fmtv(row['fx0']),
            fmtv(row['fx1']),
            fmtv(row['x2']),
            fmtv(row['error']),
        ]
        for v, w in zip(vals, col):
            line += color + f" {v:<{w}} " + C.RESET + C.GRAY + "|" + C.RESET
        print(line)

    print(sep)

def get_input(prompt, cast, validator=None, default=None):
    while True:
        try:
            raw = input(prompt).strip()
            if raw == "" and default is not None:
                return default
            value = cast(raw)
            if validator and not validator(value):
                raise ValueError
            return value
        except (ValueError, TypeError):
            print(f"  {C.RED}Input tidak valid. Coba lagi.{C.RESET}")

def main():

    fx_str = get_input(
        f"  {C.BOLD}Masukkan f(x){C.RESET} : ",
        str,
        validator=lambda s: len(s) > 0
    )

    try:
        f = parse_function(fx_str)
        _ = f(1.0)  
    except Exception as e:
        print(f"\n  {C.RED}Error parsing fungsi: {e}{C.RESET}")
        return

    print()
    x0 = get_input(f"  {C.BOLD}x₀{C.RESET} : ", float)
    x1 = get_input(f"  {C.BOLD}x₁{C.RESET} : ", float,
                   validator=lambda v: v != x0)

    tol      = 1e-5
    max_iter = get_input(f"  Iterasi maksimum  [{C.GRAY}default: 50{C.RESET}]   : ",
                         int,   validator=lambda v: v > 0, default=50)


    try:
        root, converged, table = secant_method(f, x0, x1, tol, max_iter)
    except (ValueError, ZeroDivisionError, OverflowError) as e:
        print(f"\n  {C.RED}Error: {e}{C.RESET}")
        return

    print_table(table, converged)

    print()

    print(f"  Akar            : {C.BOLD}{C.WHITE}{root:.5f}{C.RESET}")
    print(f"  f(akar)         : {C.BOLD}{C.WHITE}{f(root):.5f}{C.RESET}")
    print(f"  Error           : {C.BOLD}{C.WHITE}{table[-1]['error']:.5f}%{C.RESET}")    
    print()
if __name__ == "__main__":
    main()
