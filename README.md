<?php
require_once __DIR__.'/../config/database.php';
$data=$_GET['data'] ?? date('Y-m-d');
$s=$pdo->prepare("SELECT f.id,f.data_coleta,f.status,t.nome transportador,o.identificacao origem,d.identificacao destino,
 GROUP_CONCAT(CONCAT(p.nome,' (',fp.quantidade,')') SEPARATOR '; ') produtos
 FROM fretes f JOIN transportadores t ON t.id=f.transportador_id
 JOIN enderecos o ON o.id=f.origem_id JOIN enderecos d ON d.id=f.destino_id
 JOIN frete_produtos fp ON fp.frete_id=f.id JOIN produtos p ON p.id=fp.produto_id
 WHERE f.data_coleta=? GROUP BY f.id ORDER BY f.id");
$s->execute([$data]);
header('Content-Type: text/csv; charset=utf-8');
header('Content-Disposition: attachment; filename="relatorio-fretes-'.$data.'.csv"');
$out=fopen('php://output','w');fwrite($out, "\xEF\xBB\xBF");
fputcsv($out,['Número','Data','Transportador','Origem','Destino','Produtos','Status'],';');
while($r=$s->fetch()) fputcsv($out,[$r['id'],$r['data_coleta'],$r['transportador'],$r['origem'],$r['destino'],$r['produtos'],$r['status']],';');
fclose($out);
<?php
require_once __DIR__.'/../config/database.php';
$pageTitle='Relatórios';$current='relatorios';
$data=$_GET['data'] ?? date('Y-m-d');
$s=$pdo->prepare("SELECT f.id,f.data_coleta,f.status,t.nome transportador,o.identificacao origem,d.identificacao destino,
 GROUP_CONCAT(CONCAT(p.nome,' (',fp.quantidade,')') SEPARATOR ', ') produtos
 FROM fretes f JOIN transportadores t ON t.id=f.transportador_id
 JOIN enderecos o ON o.id=f.origem_id JOIN enderecos d ON d.id=f.destino_id
 JOIN frete_produtos fp ON fp.frete_id=f.id JOIN produtos p ON p.id=fp.produto_id
 WHERE f.data_coleta=? GROUP BY f.id ORDER BY f.id");
$s->execute([$data]);$items=$s->fetchAll();
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1>Relatório diário</h1><p class="subtitle">Filtre os fretes por data e exporte os resultados em CSV.</p></div></div>
<div class="card form-card mb-3"><form class="row g-2 align-items-end"><div class="col-md-4"><label class="form-label">Data dos fretes</label><input class="form-control" type="date" name="data" value="<?=e($data)?>"></div><div class="col-auto"><button class="btn btn-primary">Consultar</button></div><div class="col-auto"><a class="btn btn-outline-success" href="csv.php?data=<?=urlencode($data)?>">Baixar CSV</a></div></form></div>
<div class="card p-3"><div class="d-flex justify-content-between mb-3"><h5 class="fw-bold">Resultado · <?=date('d/m/Y',strtotime($data))?></h5><span class="badge-soft"><?=count($items)?> frete(s)</span></div>
<div class="table-responsive"><table class="table"><thead><tr><th>Nº</th><th>Transportador</th><th>Origem</th><th>Destino</th><th>Produtos / Quantidade</th><th>Status</th></tr></thead><tbody>
<?php if(!$items): ?><tr><td colspan="6" class="text-center text-secondary py-4">Não há fretes nesta data.</td></tr><?php endif; ?>
<?php foreach($items as $r): ?><tr><td>#<?=$r['id']?></td><td><?=e($r['transportador'])?></td><td><?=e($r['origem'])?></td><td><?=e($r['destino'])?></td><td><?=e($r['produtos'])?></td><td><?=e($r['status'])?></td></tr><?php endforeach; ?>
</tbody></table></div></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');
:root{--navy:#142b4a;--blue:#2563eb;--muted:#64748b;--bg:#f4f7fb}
*{box-sizing:border-box}body{font-family:Inter,system-ui,sans-serif;background:var(--bg);color:#172033}
.app-navbar{background:var(--navy);padding:.85rem 1.5rem;box-shadow:0 3px 16px #142b4a20}
.navbar-brand{letter-spacing:.2px}.brand-mark{display:inline-grid;place-items:center;background:#3b82f6;border-radius:9px;width:30px;height:30px;margin-right:7px}
.nav-link{color:#dbeafe!important;margin-left:.25rem;border-radius:8px;padding:.55rem .8rem!important}.nav-link:hover,.nav-link.active{background:#ffffff1c;color:white!important}
.page-wrap{padding:30px 34px;min-height:calc(100vh - 130px);max-width:1600px;margin:auto}
.page-heading{display:flex;justify-content:space-between;align-items:center;gap:16px;margin-bottom:24px;flex-wrap:wrap}
.page-heading h1{font-size:1.75rem;font-weight:800;margin:0;color:#142b4a}.subtitle{color:var(--muted);margin:.4rem 0 0}
.card{border:0;border-radius:16px;box-shadow:0 5px 22px #142b4a0b}.stat-card{padding:22px}.stat-label{font-size:.85rem;color:var(--muted);font-weight:600}.stat-value{font-size:2rem;font-weight:800;color:var(--navy);margin-top:8px}
.stat-icon{width:42px;height:42px;border-radius:12px;display:grid;place-items:center;background:#eaf1ff;color:#2563eb;font-size:1.2rem}
.btn{border-radius:9px;font-weight:600;padding:.58rem .9rem}.btn-primary{background:#2563eb;border-color:#2563eb}.btn-primary:hover{background:#1d4ed8}
.table{vertical-align:middle}.table thead th{font-size:.78rem;text-transform:uppercase;letter-spacing:.04em;color:#64748b;background:#f8fafc;border-bottom:1px solid #e7edf5;padding:14px}.table td{padding:14px;border-color:#edf1f6}
.form-card{padding:24px}.form-label{font-weight:600;font-size:.9rem}.form-control,.form-select{border-radius:9px;padding:.65rem .8rem;border-color:#dbe3ef}
.section-card{padding:22px}.empty-state{text-align:center;padding:42px;color:var(--muted)}.app-footer{padding:18px 34px;color:#8491a5;font-size:.8rem;border-top:1px solid #e5eaf1;background:white}
.badge-soft{background:#eaf1ff;color:#2455a6;padding:7px 10px;border-radius:20px;font-weight:600}
@media(max-width:700px){.page-wrap{padding:22px 15px}.app-navbar{padding:.8rem 1rem}.page-heading h1{font-size:1.45rem}}
<?php
declare(strict_types=1);

$host = 'localhost';
$db   = 'gerenciador_logistico';
$user = 'root';
$pass = '';
$charset = 'utf8mb4';

$dsn = "mysql:host={$host};dbname={$db};charset={$charset}";
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
];

try {
    $pdo = new PDO($dsn, $user, $pass, $options);
} catch (PDOException $e) {
    http_response_code(500);
    exit('Não foi possível conectar ao banco de dados. Confira o banco e as configurações em config/database.php.');
}
CREATE DATABASE IF NOT EXISTS gerenciador_logistico CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE gerenciador_logistico;

CREATE TABLE IF NOT EXISTS produtos (
 id INT AUTO_INCREMENT PRIMARY KEY,
 nome VARCHAR(120) NOT NULL,
 descricao TEXT NULL,
 peso DECIMAL(10,2) NOT NULL DEFAULT 0,
 valor DECIMAL(10,2) NOT NULL DEFAULT 0,
 created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS enderecos (
 id INT AUTO_INCREMENT PRIMARY KEY,
 identificacao VARCHAR(100) NOT NULL,
 cep VARCHAR(9) NULL,
 logradouro VARCHAR(150) NOT NULL,
 numero VARCHAR(20) NULL,
 complemento VARCHAR(100) NULL,
 bairro VARCHAR(100) NULL,
 cidade VARCHAR(100) NOT NULL,
 estado CHAR(2) NOT NULL,
 created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS transportadores (
 id INT AUTO_INCREMENT PRIMARY KEY,
 nome VARCHAR(120) NOT NULL,
 documento VARCHAR(20) NULL,
 telefone VARCHAR(30) NULL,
 email VARCHAR(120) NULL,
 tipo_veiculo VARCHAR(80) NULL,
 created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS fretes (
 id INT AUTO_INCREMENT PRIMARY KEY,
 transportador_id INT NOT NULL,
 origem_id INT NOT NULL,
 destino_id INT NOT NULL,
 data_coleta DATE NOT NULL,
 status ENUM('Pendente','Em trânsito','Entregue','Cancelado') NOT NULL DEFAULT 'Pendente',
 observacoes TEXT NULL,
 created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
 CONSTRAINT fk_frete_transportador FOREIGN KEY (transportador_id) REFERENCES transportadores(id),
 CONSTRAINT fk_frete_origem FOREIGN KEY (origem_id) REFERENCES enderecos(id),
 CONSTRAINT fk_frete_destino FOREIGN KEY (destino_id) REFERENCES enderecos(id)
);

CREATE TABLE IF NOT EXISTS frete_produtos (
 frete_id INT NOT NULL,
 produto_id INT NOT NULL,
 quantidade INT NOT NULL DEFAULT 1,
 PRIMARY KEY (frete_id, produto_id),
 CONSTRAINT fk_fp_frete FOREIGN KEY (frete_id) REFERENCES fretes(id) ON DELETE CASCADE,
 CONSTRAINT fk_fp_produto FOREIGN KEY (produto_id) REFERENCES produtos(id)
);
<?php
require_once __DIR__.'/../config/database.php';
if (session_status() === PHP_SESSION_NONE) session_start();
require_once __DIR__.'/../includes/functions.php';
$pageTitle='Endereços'; $current='enderecos';
$id=(int)($_GET['id'] ?? 0); $item=[];
if($id){ $s=$pdo->prepare("SELECT * FROM enderecos WHERE id=?");$s->execute([$id]);$item=$s->fetch() ?: []; }
if($_SERVER['REQUEST_METHOD']==='POST'){
  $values=[post('identificacao'), post('cep'), post('logradouro'), post('numero'), post('complemento'), post('bairro'), post('cidade'), post('estado')];
  if($id){ $sql="UPDATE enderecos SET identificacao=?, cep=?, logradouro=?, numero=?, complemento=?, bairro=?, cidade=?, estado=? WHERE id=?"; $pdo->prepare($sql)->execute([...$values,$id]); }
  else { $pdo->prepare("INSERT INTO enderecos (identificacao, cep, logradouro, numero, complemento, bairro, cidade, estado) VALUES (?, ?, ?, ?, ?, ?, ?, ?)")->execute($values); }
  flash('Cadastro salvo com sucesso.'); redirect('index.php');
}
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1><?= $id?'Editar':'Novo' ?> · Endereço</h1><p class="subtitle">Preencha os dados abaixo.</p></div><a class="btn btn-outline-secondary" href="index.php">Voltar</a></div>
<div class="card form-card"><form method="post" class="row g-3"><div class="col-md-6"><label class="form-label">Identificação (ex.: Depósito)</label><input class="form-control" type="text" name="identificacao" value="<?= e($item['identificacao'] ?? '') ?>" required></div>
<div class="col-md-6"><label class="form-label">CEP</label><input class="form-control" type="text" name="cep" value="<?= e($item['cep'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">Logradouro</label><input class="form-control" type="text" name="logradouro" value="<?= e($item['logradouro'] ?? '') ?>" required></div>
<div class="col-md-6"><label class="form-label">Número</label><input class="form-control" type="text" name="numero" value="<?= e($item['numero'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">Complemento</label><input class="form-control" type="text" name="complemento" value="<?= e($item['complemento'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">Bairro</label><input class="form-control" type="text" name="bairro" value="<?= e($item['bairro'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">Cidade</label><input class="form-control" type="text" name="cidade" value="<?= e($item['cidade'] ?? '') ?>" required></div>
<div class="col-md-6"><label class="form-label">UF</label><input class="form-control" type="text" name="estado" value="<?= e($item['estado'] ?? '') ?>" required></div><div class="col-12 mt-4"><button class="btn btn-primary">Salvar cadastro</button> <a class="btn btn-light" href="index.php">Cancelar</a></div></form></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
<?php
require_once __DIR__.'/../config/database.php';
$pageTitle='Endereços'; $current='enderecos';
$items=$pdo->query("SELECT * FROM enderecos ORDER BY id DESC")->fetchAll();
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1>Endereços</h1><p class="subtitle">Gerencie os registros cadastrados.</p></div><a class="btn btn-primary" href="form.php">＋ Novo cadastro</a></div>
<div class="card p-3"><div class="table-responsive"><table class="table mb-0"><thead><tr><th>ID</th><th>Identificação (ex.: Depósito)</th><th>CEP</th><th>Logradouro</th><th>Número</th><th>Complemento</th><th>Bairro</th><th>Cidade</th><th>UF</th><th>Ações</th></tr></thead><tbody>
<?php if (!$items): ?><tr><td colspan="10" class="text-center text-secondary py-4">Nenhum registro cadastrado.</td></tr><?php endif; ?>
<?php foreach($items as $item): ?><tr><td>#<?= (int)$item['id'] ?></td><td><?= e((string)$item["identificacao"]) ?></td><td><?= e((string)$item["cep"]) ?></td><td><?= e((string)$item["logradouro"]) ?></td><td><?= e((string)$item["numero"]) ?></td><td><?= e((string)$item["complemento"]) ?></td><td><?= e((string)$item["bairro"]) ?></td><td><?= e((string)$item["cidade"]) ?></td><td><?= e((string)$item["estado"]) ?></td><td><a class="btn btn-sm btn-outline-primary" href="form.php?id=<?= (int)$item['id'] ?>">Editar</a> <a class="btn btn-sm btn-outline-danger" data-confirm="Excluir este registro?" href="delete.php?id=<?= (int)$item['id'] ?>">Excluir</a></td></tr><?php endforeach; ?>
</tbody></table></div></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
<?php
require_once __DIR__.'/../config/database.php';
if (session_status() === PHP_SESSION_NONE) session_start();
require_once __DIR__.'/../includes/functions.php';
$pageTitle='Novo frete'; $current='fretes';
$id=(int)($_GET['id'] ?? 0);
$transportadores=$pdo->query("SELECT id,nome FROM transportadores ORDER BY nome")->fetchAll();
$enderecos=$pdo->query("SELECT id,identificacao,cidade,estado FROM enderecos ORDER BY identificacao")->fetchAll();
$produtos=$pdo->query("SELECT id,nome FROM produtos ORDER BY nome")->fetchAll();
if($_SERVER['REQUEST_METHOD']==='POST'){
  $transportador=(int)post('transportador_id'); $origem=(int)post('origem_id'); $destino=(int)post('destino_id');
  $data=post('data_coleta'); $status=post('status','Pendente');
  $ids=$_POST['produto_id'] ?? []; $qtds=$_POST['quantidade'] ?? [];
  if(!$transportador||!$origem||!$destino||!$data||!count($ids)){ flash('Preencha os dados do frete e adicione ao menos um produto.','warning'); redirect('form.php'); }
  $pdo->beginTransaction();
  try {
    $s=$pdo->prepare("INSERT INTO fretes(transportador_id,origem_id,destino_id,data_coleta,status,observacoes) VALUES(?,?,?,?,?,?)");
    $s->execute([$transportador,$origem,$destino,$data,$status,post('observacoes')]); $freteId=(int)$pdo->lastInsertId();
    $p=$pdo->prepare("INSERT INTO frete_produtos(frete_id,produto_id,quantidade) VALUES(?,?,?)");
    foreach($ids as $i=>$pid){ if((int)$pid>0) $p->execute([$freteId,(int)$pid,max(1,(int)($qtds[$i]??1))]); }
    $pdo->commit(); flash('Frete criado com sucesso.'); redirect('index.php');
  } catch(Throwable $e){ $pdo->rollBack(); throw $e; }
}
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1>Novo frete</h1><p class="subtitle">Relacione transportador, percurso e um ou mais produtos.</p></div><a class="btn btn-outline-secondary" href="index.php">Voltar</a></div>
<?php if(!$transportadores||count($enderecos)<2||!$produtos): ?><div class="alert alert-warning">Antes de gerar um frete, cadastre ao menos um transportador, dois endereços e um produto.</div><?php endif; ?>
<form method="post" class="card form-card">
<div class="row g-3">
<div class="col-md-4"><label class="form-label">Transportador</label><select class="form-select" name="transportador_id" required><option value="">Selecione</option><?php foreach($transportadores as $t): ?><option value="<?=$t['id']?>"><?=e($t['nome'])?></option><?php endforeach; ?></select></div>
<div class="col-md-4"><label class="form-label">Endereço de origem</label><select class="form-select" name="origem_id" required><option value="">Selecione</option><?php foreach($enderecos as $a): ?><option value="<?=$a['id']?>"><?=e($a['identificacao'].' · '.$a['cidade'].'/'.$a['estado'])?></option><?php endforeach; ?></select></div>
<div class="col-md-4"><label class="form-label">Endereço de destino</label><select class="form-select" name="destino_id" required><option value="">Selecione</option><?php foreach($enderecos as $a): ?><option value="<?=$a['id']?>"><?=e($a['identificacao'].' · '.$a['cidade'].'/'.$a['estado'])?></option><?php endforeach; ?></select></div>
<div class="col-md-4"><label class="form-label">Data da coleta</label><input class="form-control" type="date" name="data_coleta" required value="<?=date('Y-m-d')?>"></div>
<div class="col-md-4"><label class="form-label">Status</label><select class="form-select" name="status"><option>Pendente</option><option>Em trânsito</option><option>Entregue</option><option>Cancelado</option></select></div>
<div class="col-12"><hr><div class="d-flex justify-content-between align-items-center mb-2"><h5 class="fw-bold mb-0">Produtos do frete</h5><button type="button" class="btn btn-sm btn-outline-primary" data-add-product-row>＋ Adicionar produto</button></div>
<div id="product-rows"><div class="row g-2 product-row mb-2"><div class="col-md-8"><select class="form-select" name="produto_id[]" required><option value="">Selecione um produto</option><?php foreach($produtos as $p): ?><option value="<?=$p['id']?>"><?=e($p['nome'])?></option><?php endforeach; ?></select></div><div class="col-md-3"><input class="form-control" type="number" min="1" name="quantidade[]" value="1" required></div><div class="col-md-1"><button type="button" class="btn btn-outline-danger w-100" data-remove-row>×</button></div></div></div></div>
<div class="col-12"><label class="form-label">Observações</label><textarea class="form-control" name="observacoes" rows="3"></textarea></div>
<div class="col-12"><button class="btn btn-primary" <?=(!$transportadores||count($enderecos)<2||!$produtos)?'disabled':''?>>Gerar frete</button></div>
</div></form>
<?php require __DIR__.'/../includes/footer.php'; ?>
<?php
require_once __DIR__.'/../config/database.php';
$pageTitle='Fretes';$current='fretes';
$items=$pdo->query("SELECT f.*,t.nome transportador,o.identificacao origem,d.identificacao destino,
 (SELECT COUNT(*) FROM frete_produtos fp WHERE fp.frete_id=f.id) total_produtos
 FROM fretes f JOIN transportadores t ON t.id=f.transportador_id
 JOIN enderecos o ON o.id=f.origem_id JOIN enderecos d ON d.id=f.destino_id ORDER BY f.id DESC")->fetchAll();
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1>Fretes</h1><p class="subtitle">Consulte os envios e acompanhe o status das entregas.</p></div><a class="btn btn-primary" href="form.php">＋ Gerar frete</a></div>
<div class="card p-3"><div class="table-responsive"><table class="table mb-0"><thead><tr><th>Nº</th><th>Data</th><th>Transportador</th><th>Origem → Destino</th><th>Produtos</th><th>Status</th></tr></thead><tbody>
<?php if(!$items): ?><tr><td colspan="6" class="text-center text-secondary py-4">Nenhum frete cadastrado.</td></tr><?php endif; ?>
<?php foreach($items as $f): ?><tr><td>#<?=$f['id']?></td><td><?=date('d/m/Y',strtotime($f['data_coleta']))?></td><td><?=e($f['transportador'])?></td><td><?=e($f['origem'].' → '.$f['destino'])?></td><td><?=$f['total_produtos']?></td><td><span class="badge-soft"><?=e($f['status'])?></span></td></tr><?php endforeach; ?>
</tbody></table></div></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
</main>
<footer class="app-footer"><div class="container-fluid">LogiFlow · Gerenciador Logístico · Projeto acadêmico</div></footer>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
<script src="/gerenciador_logistico/assets/js/app.js"></script>
</body></html>
<?php
function e(?string $value): string {
    return htmlspecialchars((string)$value, ENT_QUOTES, 'UTF-8');
}
function redirect(string $url): never {
    header("Location: {$url}");
    exit;
}
function flash(string $message, string $type = 'success'): void {
    $_SESSION['flash'] = ['message' => $message, 'type' => $type];
}
function showFlash(): void {
    if (!empty($_SESSION['flash'])) {
        $f = $_SESSION['flash'];
        echo '<div class="alert alert-' . e($f['type']) . ' alert-dismissible fade show" role="alert">'
            . e($f['message']) .
            '<button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Fechar"></button></div>';
        unset($_SESSION['flash']);
    }
}
function post(string $key, string $default = ''): string {
    return trim((string)($_POST[$key] ?? $default));
}
<?php
if (session_status() === PHP_SESSION_NONE) session_start();
require_once __DIR__ . '/functions.php';
$pageTitle = $pageTitle ?? 'Painel';
$current = $current ?? '';
?>
<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title><?= e($pageTitle) ?> | LogiFlow</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <link href="/gerenciador_logistico/assets/css/style.css" rel="stylesheet">
</head>
<body>
<nav class="navbar navbar-expand-lg navbar-dark app-navbar">
  <div class="container-fluid">
    <a class="navbar-brand fw-bold" href="/gerenciador_logistico/index.php"><span class="brand-mark">L</span> LogiFlow</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#mainNav" aria-controls="mainNav" aria-expanded="false" aria-label="Abrir menu"><span class="navbar-toggler-icon"></span></button>
    <div class="collapse navbar-collapse" id="mainNav">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link <?= $current==='inicio'?'active':'' ?>" href="/gerenciador_logistico/index.php">Visão geral</a></li>
        <li class="nav-item"><a class="nav-link <?= $current==='produtos'?'active':'' ?>" href="/gerenciador_logistico/produtos/index.php">Produtos</a></li>
        <li class="nav-item"><a class="nav-link <?= $current==='enderecos'?'active':'' ?>" href="/gerenciador_logistico/enderecos/index.php">Endereços</a></li>
        <li class="nav-item"><a class="nav-link <?= $current==='transportadores'?'active':'' ?>" href="/gerenciador_logistico/transportadores/index.php">Transportadores</a></li>
        <li class="nav-item"><a class="nav-link <?= $current==='fretes'?'active':'' ?>" href="/gerenciador_logistico/fretes/index.php">Fretes</a></li>
        <li class="nav-item"><a class="nav-link <?= $current==='relatorios'?'active':'' ?>" href="/gerenciador_logistico/relatorios/index.php">Relatórios</a></li>
      </ul>
    </div>
  </div>
</nav>
<main class="container-fluid page-wrap">
<?php showFlash(); ?>
<?php
require_once __DIR__.'/../config/database.php';
if (session_status() === PHP_SESSION_NONE) session_start();
require_once __DIR__.'/../includes/functions.php';
$id=(int)($_GET['id'] ?? 0);
try { $pdo->prepare("DELETE FROM produtos WHERE id=?")->execute([$id]); flash('Registro excluído.'); }
catch(PDOException $e) { flash('Não foi possível excluir: o registro pode estar vinculado a um frete.','warning'); }
redirect('index.php');
<?php
require_once __DIR__.'/../config/database.php';
if (session_status() === PHP_SESSION_NONE) session_start();
require_once __DIR__.'/../includes/functions.php';
$pageTitle='Produtos'; $current='produtos';
$id=(int)($_GET['id'] ?? 0); $item=[];
if($id){ $s=$pdo->prepare("SELECT * FROM produtos WHERE id=?");$s->execute([$id]);$item=$s->fetch() ?: []; }
if($_SERVER['REQUEST_METHOD']==='POST'){
  $values=[post('nome'), post('descricao'), post('peso'), post('valor')];
  if($id){ $sql="UPDATE produtos SET nome=?, descricao=?, peso=?, valor=? WHERE id=?"; $pdo->prepare($sql)->execute([...$values,$id]); }
  else { $pdo->prepare("INSERT INTO produtos (nome, descricao, peso, valor) VALUES (?, ?, ?, ?)")->execute($values); }
  flash('Cadastro salvo com sucesso.'); redirect('index.php');
}
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1><?= $id?'Editar':'Novo' ?> · Produto</h1><p class="subtitle">Preencha os dados abaixo.</p></div><a class="btn btn-outline-secondary" href="index.php">Voltar</a></div>
<div class="card form-card"><form method="post" class="row g-3"><div class="col-md-6"><label class="form-label">Nome do produto</label><input class="form-control" type="text" name="nome" value="<?= e($item['nome'] ?? '') ?>" required></div>
<div class="col-md-6"><label class="form-label">Descrição</label><input class="form-control" type="text" name="descricao" value="<?= e($item['descricao'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">Peso (kg)</label><input class="form-control" type="number" name="peso" value="<?= e($item['peso'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">Valor (R$)</label><input class="form-control" type="number" name="valor" value="<?= e($item['valor'] ?? '') ?>" ></div><div class="col-12 mt-4"><button class="btn btn-primary">Salvar cadastro</button> <a class="btn btn-light" href="index.php">Cancelar</a></div></form></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
<?php
require_once __DIR__.'/../config/database.php';
$pageTitle='Produtos'; $current='produtos';
$items=$pdo->query("SELECT * FROM produtos ORDER BY id DESC")->fetchAll();
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1>Produtos</h1><p class="subtitle">Gerencie os registros cadastrados.</p></div><a class="btn btn-primary" href="form.php">＋ Novo cadastro</a></div>
<div class="card p-3"><div class="table-responsive"><table class="table mb-0"><thead><tr><th>ID</th><th>Nome do produto</th><th>Descrição</th><th>Peso (kg)</th><th>Valor (R$)</th><th>Ações</th></tr></thead><tbody>
<?php if (!$items): ?><tr><td colspan="6" class="text-center text-secondary py-4">Nenhum registro cadastrado.</td></tr><?php endif; ?>
<?php foreach($items as $item): ?><tr><td>#<?= (int)$item['id'] ?></td><td><?= e((string)$item["nome"]) ?></td><td><?= e((string)$item["descricao"]) ?></td><td><?= e((string)$item["peso"]) ?></td><td><?= e((string)$item["valor"]) ?></td><td><a class="btn btn-sm btn-outline-primary" href="form.php?id=<?= (int)$item['id'] ?>">Editar</a> <a class="btn btn-sm btn-outline-danger" data-confirm="Excluir este registro?" href="delete.php?id=<?= (int)$item['id'] ?>">Excluir</a></td></tr><?php endforeach; ?>
</tbody></table></div></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
<?php
require_once __DIR__.'/../config/database.php';
$data=$_GET['data'] ?? date('Y-m-d');
$s=$pdo->prepare("SELECT f.id,f.data_coleta,f.status,t.nome transportador,o.identificacao origem,d.identificacao destino,
 GROUP_CONCAT(CONCAT(p.nome,' (',fp.quantidade,')') SEPARATOR '; ') produtos
 FROM fretes f JOIN transportadores t ON t.id=f.transportador_id
 JOIN enderecos o ON o.id=f.origem_id JOIN enderecos d ON d.id=f.destino_id
 JOIN frete_produtos fp ON fp.frete_id=f.id JOIN produtos p ON p.id=fp.produto_id
 WHERE f.data_coleta=? GROUP BY f.id ORDER BY f.id");
$s->execute([$data]);
header('Content-Type: text/csv; charset=utf-8');
header('Content-Disposition: attachment; filename="relatorio-fretes-'.$data.'.csv"');
$out=fopen('php://output','w');fwrite($out, "\xEF\xBB\xBF");
fputcsv($out,['Número','Data','Transportador','Origem','Destino','Produtos','Status'],';');
while($r=$s->fetch()) fputcsv($out,[$r['id'],$r['data_coleta'],$r['transportador'],$r['origem'],$r['destino'],$r['produtos'],$r['status']],';');
fclose($out);
<?php
require_once __DIR__.'/../config/database.php';
$pageTitle='Relatórios';$current='relatorios';
$data=$_GET['data'] ?? date('Y-m-d');
$s=$pdo->prepare("SELECT f.id,f.data_coleta,f.status,t.nome transportador,o.identificacao origem,d.identificacao destino,
 GROUP_CONCAT(CONCAT(p.nome,' (',fp.quantidade,')') SEPARATOR ', ') produtos
 FROM fretes f JOIN transportadores t ON t.id=f.transportador_id
 JOIN enderecos o ON o.id=f.origem_id JOIN enderecos d ON d.id=f.destino_id
 JOIN frete_produtos fp ON fp.frete_id=f.id JOIN produtos p ON p.id=fp.produto_id
 WHERE f.data_coleta=? GROUP BY f.id ORDER BY f.id");
$s->execute([$data]);$items=$s->fetchAll();
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1>Relatório diário</h1><p class="subtitle">Filtre os fretes por data e exporte os resultados em CSV.</p></div></div>
<div class="card form-card mb-3"><form class="row g-2 align-items-end"><div class="col-md-4"><label class="form-label">Data dos fretes</label><input class="form-control" type="date" name="data" value="<?=e($data)?>"></div><div class="col-auto"><button class="btn btn-primary">Consultar</button></div><div class="col-auto"><a class="btn btn-outline-success" href="csv.php?data=<?=urlencode($data)?>">Baixar CSV</a></div></form></div>
<div class="card p-3"><div class="d-flex justify-content-between mb-3"><h5 class="fw-bold">Resultado · <?=date('d/m/Y',strtotime($data))?></h5><span class="badge-soft"><?=count($items)?> frete(s)</span></div>
<div class="table-responsive"><table class="table"><thead><tr><th>Nº</th><th>Transportador</th><th>Origem</th><th>Destino</th><th>Produtos / Quantidade</th><th>Status</th></tr></thead><tbody>
<?php if(!$items): ?><tr><td colspan="6" class="text-center text-secondary py-4">Não há fretes nesta data.</td></tr><?php endif; ?>
<?php foreach($items as $r): ?><tr><td>#<?=$r['id']?></td><td><?=e($r['transportador'])?></td><td><?=e($r['origem'])?></td><td><?=e($r['destino'])?></td><td><?=e($r['produtos'])?></td><td><?=e($r['status'])?></td></tr><?php endforeach; ?>
</tbody></table></div></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
<?php
require_once __DIR__.'/../config/database.php';
if (session_status() === PHP_SESSION_NONE) session_start();
require_once __DIR__.'/../includes/functions.php';
$id=(int)($_GET['id'] ?? 0);
try { $pdo->prepare("DELETE FROM transportadores WHERE id=?")->execute([$id]); flash('Registro excluído.'); }
catch(PDOException $e) { flash('Não foi possível excluir: o registro pode estar vinculado a um frete.','warning'); }
redirect('index.php');
<?php
require_once __DIR__.'/../config/database.php';
if (session_status() === PHP_SESSION_NONE) session_start();
require_once __DIR__.'/../includes/functions.php';
$pageTitle='Transportadores'; $current='transportadores';
$id=(int)($_GET['id'] ?? 0); $item=[];
if($id){ $s=$pdo->prepare("SELECT * FROM transportadores WHERE id=?");$s->execute([$id]);$item=$s->fetch() ?: []; }
if($_SERVER['REQUEST_METHOD']==='POST'){
  $values=[post('nome'), post('documento'), post('telefone'), post('email'), post('tipo_veiculo')];
  if($id){ $sql="UPDATE transportadores SET nome=?, documento=?, telefone=?, email=?, tipo_veiculo=? WHERE id=?"; $pdo->prepare($sql)->execute([...$values,$id]); }
  else { $pdo->prepare("INSERT INTO transportadores (nome, documento, telefone, email, tipo_veiculo) VALUES (?, ?, ?, ?, ?)")->execute($values); }
  flash('Cadastro salvo com sucesso.'); redirect('index.php');
}
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1><?= $id?'Editar':'Novo' ?> · Transportadore</h1><p class="subtitle">Preencha os dados abaixo.</p></div><a class="btn btn-outline-secondary" href="index.php">Voltar</a></div>
<div class="card form-card"><form method="post" class="row g-3"><div class="col-md-6"><label class="form-label">Nome / Razão social</label><input class="form-control" type="text" name="nome" value="<?= e($item['nome'] ?? '') ?>" required></div>
<div class="col-md-6"><label class="form-label">CPF / CNPJ</label><input class="form-control" type="text" name="documento" value="<?= e($item['documento'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">Telefone</label><input class="form-control" type="tel" name="telefone" value="<?= e($item['telefone'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">E-mail</label><input class="form-control" type="email" name="email" value="<?= e($item['email'] ?? '') ?>" ></div>
<div class="col-md-6"><label class="form-label">Tipo de veículo</label><input class="form-control" type="text" name="tipo_veiculo" value="<?= e($item['tipo_veiculo'] ?? '') ?>" ></div><div class="col-12 mt-4"><button class="btn btn-primary">Salvar cadastro</button> <a class="btn btn-light" href="index.php">Cancelar</a></div></form></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
<?php
require_once __DIR__.'/../config/database.php';
$pageTitle='Transportadores'; $current='transportadores';
$items=$pdo->query("SELECT * FROM transportadores ORDER BY id DESC")->fetchAll();
require __DIR__.'/../includes/header.php';
?>
<div class="page-heading"><div><h1>Transportadores</h1><p class="subtitle">Gerencie os registros cadastrados.</p></div><a class="btn btn-primary" href="form.php">＋ Novo cadastro</a></div>
<div class="card p-3"><div class="table-responsive"><table class="table mb-0"><thead><tr><th>ID</th><th>Nome / Razão social</th><th>CPF / CNPJ</th><th>Telefone</th><th>E-mail</th><th>Tipo de veículo</th><th>Ações</th></tr></thead><tbody>
<?php if (!$items): ?><tr><td colspan="7" class="text-center text-secondary py-4">Nenhum registro cadastrado.</td></tr><?php endif; ?>
<?php foreach($items as $item): ?><tr><td>#<?= (int)$item['id'] ?></td><td><?= e((string)$item["nome"]) ?></td><td><?= e((string)$item["documento"]) ?></td><td><?= e((string)$item["telefone"]) ?></td><td><?= e((string)$item["email"]) ?></td><td><?= e((string)$item["tipo_veiculo"]) ?></td><td><a class="btn btn-sm btn-outline-primary" href="form.php?id=<?= (int)$item['id'] ?>">Editar</a> <a class="btn btn-sm btn-outline-danger" data-confirm="Excluir este registro?" href="delete.php?id=<?= (int)$item['id'] ?>">Excluir</a></td></tr><?php endforeach; ?>
</tbody></table></div></div>
<?php require __DIR__.'/../includes/footer.php'; ?>
# LogiFlow — Gerenciador Logístico
<?php
require_once __DIR__.'/config/database.php';
$pageTitle='Visão geral'; $current='inicio';
$counts=[];
foreach (['produtos','enderecos','transportadores','fretes'] as $t) {
  $counts[$t]=(int)$pdo->query("SELECT COUNT(*) FROM {$t}")->fetchColumn();
}
$recent=$pdo->query("SELECT f.id,f.data_coleta,f.status,t.nome AS transportador,o.cidade AS origem,d.cidade AS destino
 FROM fretes f JOIN transportadores t ON t.id=f.transportador_id
 JOIN enderecos o ON o.id=f.origem_id JOIN enderecos d ON d.id=f.destino_id
 ORDER BY f.id DESC LIMIT 5")->fetchAll();
require __DIR__.'/includes/header.php';
?>
<div class="page-heading"><div><h1>Visão geral</h1><p class="subtitle">Acompanhe os principais dados da sua operação logística.</p></div><a class="btn btn-primary" href="/gerenciador_logistico/fretes/form.php">＋ Novo frete</a></div>
<div class="row g-3 mb-4">
<?php foreach ([['Produtos','produtos','▣'],['Endereços','enderecos','⌖'],['Transportadores','transportadores','▤'],['Fretes','fretes','⇄']] as [$label,$key,$icon]): ?>
<div class="col-6 col-xl-3"><div class="card stat-card"><div class="d-flex justify-content-between align-items-center"><span class="stat-label"><?= $label ?></span><span class="stat-icon"><?= $icon ?></span></div><div class="stat-value"><?= $counts[$key] ?></div></div></div>
<?php endforeach; ?>
</div>
<div class="card section-card"><div class="d-flex justify-content-between align-items-center mb-3"><div><h5 class="fw-bold mb-1">Fretes recentes</h5><div class="text-secondary small">Últimos registros cadastrados</div></div><a href="/gerenciador_logistico/fretes/index.php" class="btn btn-outline-primary btn-sm">Ver todos</a></div>
<?php if (!$recent): ?><div class="empty-state">Nenhum frete cadastrado. Comece criando os cadastros básicos e gere seu primeiro frete.</div>
<?php else: ?><div class="table-responsive"><table class="table"><thead><tr><th>Nº</th><th>Data</th><th>Transportador</th><th>Trajeto</th><th>Status</th></tr></thead><tbody>
<?php foreach($recent as $r): ?><tr><td>#<?= (int)$r['id'] ?></td><td><?= date('d/m/Y',strtotime($r['data_coleta'])) ?></td><td><?= e($r['transportador']) ?></td><td><?= e($r['origem']) ?> → <?= e($r['destino']) ?></td><td><span class="badge-soft"><?= e($r['status']) ?></span></td></tr><?php endforeach; ?>
</tbody></table></div><?php endif; ?></div>
<?php require __DIR__.'/includes/footer.php'; ?>


Projeto acadêmico desenvolvido para a atividade prática orientada da disciplina **Fundamentos de Programação para Internet (UNIPAR)**.

## Funcionalidades
- Cadastro, edição, listagem e exclusão de produtos.
- Cadastro, edição, listagem e exclusão de endereços de origem e destino.
- Cadastro, edição, listagem e exclusão de transportadores.
- Geração de fretes relacionando transportador, origem, destino e vários produtos.
- Relatório de fretes por dia com exportação em CSV.
- Interface responsiva com Bootstrap 5.

## Tecnologias
PHP 8+, MySQL, PDO, HTML5, CSS3, JavaScript e Bootstrap 5.

## Como executar no XAMPP
1. Instale e abra o XAMPP. Inicie **Apache** e **MySQL**.
2. Copie a pasta `gerenciador_logistico` para `C:\xampp\htdocs\`.
3. Acesse `http://localhost/phpmyadmin`.
4. Importe o arquivo `database/schema.sql` (ele cria o banco `gerenciador_logistico` e as tabelas).
5. Confira usuário e senha do MySQL em `config/database.php` (padrão XAMPP: usuário `root` e senha vazia).
6. Abra `http://localhost/gerenciador_logistico/`.

## Ordem sugerida de uso
1. Cadastre produtos.
2. Cadastre pelo menos dois endereços (origem e destino).
3. Cadastre um transportador.
4. Acesse **Fretes > Gerar frete**, escolha o transportador, os endereços e adicione um ou mais produtos.
5. Em **Relatórios**, escolha a data e clique em **Baixar CSV**.

## Estrutura
- `config/`: conexão com banco de dados.
- `database/`: script SQL de criação do banco.
- `includes/`: cabeçalho, rodapé e funções comuns.
- `produtos/`, `enderecos/`, `transportadores/`: módulos de cadastro.
- `fretes/`: criação e consulta de fretes.
- `relatorios/`: consulta diária e exportação CSV.
- `assets/`: estilos e JavaScript.

## Modelo de dados
Um frete pertence a um transportador e possui endereço de origem e destino. A relação entre fretes e produtos é muitos-para-muitos por meio da tabela `frete_produtos`, permitindo incluir vários produtos em um mesmo frete.

## Versionamento Git
Exemplo de sequência de comandos:
```bash
git init
git add .
git commit -m "feat: estrutura inicial do gerenciador logístico"
git add .
git commit -m "feat: implementa cadastros e geração de fretes"
git add .
git commit -m "feat: adiciona relatório diário e documentação"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/gerenciador-logistico.git
git push -u origin main
```

## Observações
Este projeto é uma base acadêmica funcional para execução local. Antes de publicar em produção, adicione autenticação, proteção CSRF, validações adicionais, controle de permissões e configuração segura de credenciais.
