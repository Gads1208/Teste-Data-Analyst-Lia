# Teste-Data-Analyst-Lia
Questão 1.
Dadas as 3 tabelas:

- students: (id int, name text, enrolled_at date, course_id text)
- courses: (id int, name text, price numeric, school_id text)
- schools: (id int, name text)
Considere que todos alunos se matricularam nos respectivos cursos e que price é o valor da matrícula, pago por cada aluno.



a. Escreva uma consulta PostgreSQL para obter, por nome da escola e por dia, a quantidade de alunos matriculados e o valor total das matrículas, tendo como restrição os cursos que começam com a palavra “data”. Ordene o resultado do dia mais recente para o mais antigo.

SELECT 
    s.name AS school_name,
    st.enrolled_at AS enrollment_date,
    COUNT(st.id) AS total_students,
    SUM(c.price) AS total_revenue
FROM students st
JOIN courses c ON st.course_id = c.id
JOIN schools s ON c.school_id = s.id
WHERE c.name ILIKE 'data%'  -- Filtra apenas cursos que começam com "data"
GROUP BY s.name, st.enrolled_at
ORDER BY enrollment_date DESC;

b.Utilizando a resposta do item a, escreva uma consulta para obter, por escola e por dia, a soma acumulada, a média móvel 7 dias e a média móvel 30 dias da quantidade de alunos.

WITH enrollment_data AS (
    SELECT 
        s.name AS school_name,
        st.enrolled_at AS enrollment_date,
        COUNT(st.id) AS total_students
    FROM students st
    JOIN courses c ON st.course_id = c.id
    JOIN schools s ON c.school_id = s.id
    WHERE c.name ILIKE 'data%'
    GROUP BY s.name, st.enrolled_at
)
SELECT 
    school_name,
    enrollment_date,
    total_students,
    SUM(total_students) OVER (
        PARTITION BY school_name 
        ORDER BY enrollment_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_sum,
    AVG(total_students) OVER (
        PARTITION BY school_name 
        ORDER BY enrollment_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7_days,
    AVG(total_students) OVER (
        PARTITION BY school_name 
        ORDER BY enrollment_date 
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS moving_avg_30_days
FROM enrollment_data
ORDER BY enrollment_date DESC;

Questão 2.
Para cada departamento, realize uma consulta em PostgresSQL que mostre o nome do departamento, a quantidade de empregados, a média salarial, o maior e o menor salários. Ordene o resultado pela maior média salarial.

Dicas: você pode usar a função COALESCE(value , 0) para substituir um valor nulo por zero e pode usar a função ROUND(value, 2) para mostrar valores com duas casas decimais.

SELECT 
    d.nome AS departamento,
    COUNT(e.matr) AS qtd_empregados,
    ROUND(COALESCE(AVG(v.valor), 0), 2) AS media_salarial,
    ROUND(COALESCE(MAX(v.valor), 0), 2) AS maior_salario,
    ROUND(COALESCE(MIN(v.valor), 0), 2) AS menor_salario
FROM departamento d
LEFT JOIN divisao div ON d.cod_dep = div.cod_dep
LEFT JOIN empregado e ON div.cod_divisao = e.lotacao_div
LEFT JOIN emp_venc ev ON e.matr = ev.matr
LEFT JOIN vencimento v ON ev.cod_venc = v.cod_venc
GROUP BY d.nome
ORDER BY media_salarial DESC;

