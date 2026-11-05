# Notas de otimização – build do Breno

Arquivo compartilhado entre o Claude e o ChatGPT Codex. Quem mexer no código anota aqui o que fez,
o que mediu e o que deixou pendente. Escreva em português; o código e os commits seguem o padrão do
projeto (inglês).

## Objetivo

Fazer o **The Legend of Zelda: Ocarina of Time** (Switch 2, program id `0400486024DEC000`) rodar
o melhor possível no PC do Breno, sem quebrar a imagem.

## PC alvo

- CPU: AMD Ryzen 5 5500X3D (Zen 3, 6 núcleos / 12 threads, AVX2, sem AVX-512)
- GPU: NVIDIA GeForce RTX 5060 (Blackwell), Vulkan
- RAM: 16 GB
- Windows 11, build MSVC (a PGO/MinGW não tem decodificador AV1: o menu do Zelda fica preto)

## Ponto de partida

- Branch `breno-oot-otimizado` = `Frizti/Lichii` branch `oot-gather-fix` (commit `cafd28d`), a melhor
  build testada até agora (correções de gather com comparação de profundidade e per-tap F16, patch
  de resolução máxima, fim dos clarões, névoa e vegetação).
- Configuração do jogo usada nos testes: Vulkan, escala 1x, GPU Normal, shaders assíncronos,
  FSR3 Frame Generation 2x, DLSS do emulador desligado.

## O que os logs mostram

- 2026-11-05, build Frizti, CPU em Accurate, FG ligado, patch de resolução máxima:
  `last 600 presents: frame ~53 ms` (~19 fps reais), `blocked on the GPU fence ~3.8 ms`.
  A GPU espera pouco: o gargalo passou a ser a CPU / thread do emulador.
- Antes (build Selevet, sem patch de resolução, sem FG): frame ~34.5 ms, GPU fence ~18.8 ms
  (gargalo na GPU).

## Feito

- [Claude] Opção `LICHII_MSVC_AVX2` no CMake (`/arch:AVX2` no MSVC) e entrada `msvc_avx2` no
  workflow `release-binaries.yml`, junto com `windows_only` (compila só o Windows MSVC).

## Próximos passos

1. Medir a build AVX2 contra a build original do Frizti no mesmo lugar do jogo (mesmo save, mesma
   configuração). Comparar `last 600 presents` no log.
2. Rodar com `EDEN_NX2_GPU_PROFILE=1` (tempo da thread da GPU por zona) e
   `EDEN_DKB_FRAME_TRACE=1` (intervalo por quadro, tempo de CPU e de fence) para achar onde o tempo
   vai. Ver `AGENTS.md` para as outras ferramentas de diagnóstico.
3. Atacar a zona mais cara que aparecer. Não chutar: toda mudança precisa de medição antes/depois.
4. Testar CPU Auto x Accurate e `EDEN_NX2_FRAMES_AHEAD=2` e anotar o efeito.

## Como compilar

GitHub → Actions → `release-binaries` → Run workflow, escolher a branch, marcar
`windows_only` e `msvc_avx2`, desmarcar `publish_release`. O artefato
`lichii-windows-x86_64-<commit>` sai em ~45–60 min (menos quando o cache já existe).

## Regras de convivência

- Cada um trabalha na própria branch (`claude/...`, `codex/...`) e abre um PR para
  `breno-oot-otimizado`. O outro revisa antes de juntar.
- Uma mudança por commit, com a medição na mensagem ou aqui.
- Antes de escrever uma correção, olhar se o upstream (`zortos293/Lichii`, `Djihads80/Eglantine`)
  ou o Frizti já têm (ver `AGENTS.md`).
