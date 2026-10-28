
import streamlit as st
import sympy as sp
from sympy.parsing.sympy_parser import parse_expr
from itertools import product
import re

st.title("LogicLab")
st.write("Aplikasi Evaluasi Logika Matematika")

def parse_logic(text):
    names = set(re.findall(r"\b[A-Za-z]\w*\b", text))
    reserved = {"True", "False"}
    symbols = {
        name: sp.Symbol(name)
        for name in names if name not in reserved
    }
    expr = parse_expr(text, local_dict=symbols)
    if expr not in (sp.true, sp.false) and expr not in symbols.values():
        if not isinstance(expr, sp.logic.boolalg.Boolean):
            raise ValueError("Input bukan ekspresi logika.")
    return expr

def make_table(expr):
    variables = sorted(expr.free_symbols, key=str)
    rows = []
    for values in product([False, True],
                          repeat=len(variables)):
        result = expr.subs(dict(zip(
            variables, values
        )))
        rows.append({
            **{str(v): val for v, val
               in zip(variables, values)},
            "Hasil": bool(result)
        })
    return rows

st.subheader("1. Tabel Kebenaran")
user_expr = st.text_input(
    "Masukkan ekspresi",
    value="p & q"
)

if st.button("Evaluasi Ekspresi"):
    try:
        expr = parse_logic(user_expr)
        rows = make_table(expr)
        st.dataframe(rows, use_container_width=True)

        results = [row["Hasil"] for row in rows]
        if all(results):
            st.success("Tautologi")
        elif not any(results):
            st.error("Kontradiksi")
        else:
            st.info("Kontingensi")
    except Exception as e:
        st.error(f"Input tidak valid: {e}")

st.subheader("2. Validitas Argumen")
premise_text = st.text_input(
    "Premis, pisahkan dengan titik koma",
    value="p >> q; p"
)
conclusion_text = st.text_input(
    "Kesimpulan",
    value="q"
)

if st.button("Uji Validitas"):
    try:
        premises = [
            parse_logic(x.strip())
            for x in premise_text.split(";")
            if x.strip()
        ]
        conclusion = parse_logic(conclusion_text)

        counterexample = sp.satisfiable(
            sp.And(*premises, sp.Not(conclusion))
        )

        if counterexample is False:
            st.success("Argumen VALID")
        else:
            st.error("Argumen TIDAK VALID")
            st.write("Contoh kondisi:", counterexample)
    except Exception as e:
        st.error(f"Input tidak valid: {e}")
