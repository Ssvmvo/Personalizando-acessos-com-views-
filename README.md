# Personalizando-acessos-com-views-
Neste desafio você irá criar visões para os seguintes cenários   Número de empregados por departamento e localidade   Lista de departamentos e seus gerentes   Projetos com maior número de empregados (ex: por ordenação desc)   Lista de projetos, departamentos e gerentes   Quais empregados possuem dependentes e se são gerentes    

Parte 1: Personalizando Acessos com Views
Views para os Cenários
Número de empregados por departamento e localidade

sql
Copiar código
CREATE VIEW vw_empregados_por_departamento_localidade AS
SELECT d.nome AS departamento, e.localidade, COUNT(e.id) AS total_empregados
FROM empregados e
JOIN departamentos d ON e.departamento_id = d.id
GROUP BY d.nome, e.localidade;
Lista de departamentos e seus gerentes

sql
Copiar código
CREATE VIEW vw_departamentos_gerentes AS
SELECT d.nome AS departamento, g.nome AS gerente
FROM departamentos d
JOIN empregados g ON d.gerente_id = g.id;
Projetos com maior número de empregados

sql
Copiar código
CREATE VIEW vw_projetos_maior_numero_empregados AS
SELECT p.nome AS projeto, COUNT(ep.empregado_id) AS total_empregados
FROM projetos p
JOIN empregado_projeto ep ON p.id = ep.projeto_id
GROUP BY p.nome
ORDER BY total_empregados DESC;
Lista de projetos, departamentos e gerentes

sql
Copiar código
CREATE VIEW vw_projetos_departamentos_gerentes AS
SELECT p.nome AS projeto, d.nome AS departamento, g.nome AS gerente
FROM projetos p
JOIN departamentos d ON p.departamento_id = d.id
JOIN empregados g ON d.gerente_id = g.id;
Empregados com dependentes e se são gerentes

sql
Copiar código
CREATE VIEW vw_empregados_dependentes_gerentes AS
SELECT e.nome AS empregado, IF(COUNT(d.id) > 0, 'Sim', 'Não') AS possui_dependentes, 
       IF(e.id = g.gerente_id, 'Sim', 'Não') AS e_gerente
FROM empregados e
LEFT JOIN dependentes d ON e.id = d.empregado_id
LEFT JOIN departamentos g ON e.id = g.gerente_id
GROUP BY e.nome;
Criação de Usuários e Definição de Permissões
Criação de usuário para gerente:

sql
Copiar código
CREATE USER 'gerente'@'localhost' IDENTIFIED BY 'senha_secreta';
GRANT SELECT ON vw_empregados_por_departamento_localidade TO 'gerente'@'localhost';
GRANT SELECT ON vw_departamentos_gerentes TO 'gerente'@'localhost';
Criação de usuário para empregado (sem acesso a informações de departamentos e gerentes):

sql
Copiar código
CREATE USER 'empregado'@'localhost' IDENTIFIED BY 'senha_secreta';
GRANT SELECT ON vw_projetos_maior_numero_empregados TO 'empregado'@'localhost';
GRANT SELECT ON vw_projetos_departamentos_gerentes TO 'empregado'@'localhost';
Parte 2: Criando Gatilhos para o Cenário de E-commerce
Triggers de Remoção e Atualização
Trigger Before Delete: Garantir que as informações de usuários excluídos sejam armazenadas em uma tabela de backup.

sql
Copiar código
CREATE TRIGGER before_delete_usuario
BEFORE DELETE ON usuarios
FOR EACH ROW
BEGIN
   INSERT INTO usuarios_backup (id, nome, email, data_exclusao)
   VALUES (OLD.id, OLD.nome, OLD.email, NOW());
END;
Trigger Before Update: Controlar inserções de novos colaboradores e atualizações de salário.

sql
Copiar código
CREATE TRIGGER before_update_salario
BEFORE UPDATE ON empregados
FOR EACH ROW
BEGIN
   IF OLD.salario <> NEW.salario THEN
      INSERT INTO historico_salario (empregado_id, salario_antigo, salario_novo, data_atualizacao)
      VALUES (OLD.id, OLD.salario, NEW.salario, NOW());
   END IF;
END;
