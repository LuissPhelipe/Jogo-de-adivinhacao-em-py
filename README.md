import random
import os
import time # Import the time module

RANKING_ARQUIVO = 'ranking.txt'
TEMPO_LIMITE_PALPITE = 15 # Define a time limit for each guess in seconds
MAX_TENTATIVAS = 5

# Define difficulty order for sorting
DIFFICULTY_ORDER = {
    "Fácil": 1,
    "Médio": 2,
    "Difícil": 3
}

def carregar_ranking():
  """Loads the top 5 ranking (username, score, difficulty) from the file, if it exists."""
  ranking = []
  if os.path.exists(RANKING_ARQUIVO):
    with open(RANKING_ARQUIVO, 'r') as f:
      for line in f:
        try:
          # Expecting format: Username,Score,Difficulty
          parts = line.strip().split(',')
          if len(parts) == 3: # Ensure we have all three parts
            username, score_str, difficulty_str = parts
            ranking.append((username, int(score_str), difficulty_str))
          elif len(parts) == 2: # Handle old format gracefully (assign default difficulty)
            username, score_str = parts
            ranking.append((username, int(score_str), "Médio")) # Assign a default difficulty for old entries
        except ValueError:
          # Ignore malformed lines
          pass
  # Sort by difficulty (ascending) and then by score (descending)
  # Ensure difficulty is in DIFFICULTY_ORDER, otherwise default to a high value
  ranking.sort(key=lambda x: (DIFFICULTY_ORDER.get(x[2], 99), -x[1]))
  return ranking[:5] # Return only the top 5 entries

def salvar_ranking(ranking):
  """Saves the current top 5 ranking to the file."""
  # Ensure only top 5 are saved, and they are sorted by difficulty and score
  # Sort by difficulty (ascending) and then by score (descending)
  # Ensure difficulty is in DIFFICULTY_ORDER, otherwise default to a high value
  ranking_to_save = sorted(ranking, key=lambda x: (DIFFICULTY_ORDER.get(x[2], 99), -x[1]))[:5]
  with open(RANKING_ARQUIVO, 'w') as f:
    for username, score, difficulty in ranking_to_save:
      f.write(f"{username},{score},{difficulty}\n")

def resetar_ranking():
  """Clears the ranking file and resets the global ranking list."""
  global ranking_global
  if os.path.exists(RANKING_ARQUIVO):
    os.remove(RANKING_ARQUIVO)
  ranking_global = []
  print("Ranking foi reiniciado com sucesso!")

def obter_nome_jogador():
  """Gets the player's username."""
  nome_jogador = input("Digite seu nome de usuário para o ranking: ")
  while not nome_jogador.strip():
      print("O nome de usuário não pode estar vazio.")
      nome_jogador = input("Digite seu nome de usuário para o ranking: ")
  return nome_jogador

def exibir_ranking(ranking, title="Ranking Atual"):
  """Displays the current ranking."""
  print(f"\n--- {title} ---")
  if ranking:
    for i, (username, score, difficulty) in enumerate(ranking):
      print(f"{i+1}. {username} ({difficulty}): {score} pontos")
  else:
    print("Nenhum recorde ainda. Seja o primeiro!")
  print("---------------------\n")

def escolher_nivel_dificuldade():
  """Allows the player to choose a difficulty level and returns the number range and difficulty name."""
  print("Escolha o nível de dificuldade:")
  print("1. Fácil (1 a 50)")
  print("2. Médio (1 a 100)")
  print("3. Difícil (1 a 500)")

  while True:
    try:
      nivel = int(input("Digite o número do nível desejado: "))
      if nivel == 1:
        return 1, 50, "Fácil"
      elif nivel == 2:
        return 1, 100, "Médio"
      elif nivel == 3:
        return 1, 500, "Difícil"
      else:
        print("Nível inválido. Por favor, escolha 1, 2 ou 3.")
    except ValueError:
      print("Entrada inválida. Digite um número para o nível.")

def obter_palpite_com_tempo(tentativa_num, max_tentativas):
  """Gets a timed guess from the player and returns the guess and if it was on time."""
  start_time = time.time()
  palpite_input = input(f"Tentativa {tentativa_num}/{max_tentativas} - Digite seu palpite (Você tem {TEMPO_LIMITE_PALPITE}s): ")
  elapsed_time = time.time() - start_time

  if elapsed_time > TEMPO_LIMITE_PALPITE:
    print(f"Você demorou demais para responder! ({int(elapsed_time)}s)")
    return None, False # Indicate no valid guess and timed out

  try:
    return int(palpite_input), True # Return the integer guess and that it was on time
  except ValueError:
    print("Entrada inválida. Por favor, digite um número inteiro.")
    return None, True # Indicate no valid guess but it was on time (for attempt count)

def jogar_adivinhacao():
  global ranking_global # To modify the global ranking list

  print("\n--- Jogo de Adivinhação ---")

  nome_jogador = obter_nome_jogador()
  exibir_ranking(ranking_global)

  limite_inferior, limite_superior, dificuldade_atual = escolher_nivel_dificuldade() # Get difficulty name
  numero_secreto = random.randint(limite_inferior, limite_superior)

  tentativas_restantes = MAX_TENTATIVAS
  tentativas_feitas = 0
  dica_par_impar_dada = False
  acertou = False

  print(f"Tente adivinhar o número entre {limite_inferior} e {limite_superior}. Você tem {tentativas_restantes} tentativas.")
  print(f"Você tem {TEMPO_LIMITE_PALPITE} segundos para cada palpite!\n")

  while tentativas_restantes > 0 and not acertou:
    tentativas_feitas += 1
    palpite, on_time = obter_palpite_com_tempo(tentativas_feitas, MAX_TENTATIVAS)

    if not on_time:
      print("Uma tentativa foi perdida devido ao tempo.")
      tentativas_restantes -= 1
    elif palpite is None:
      # Invalid input but on time (count as an attempt)
      tentativas_restantes -= 1
    else:
      # Valid guess, process it
      if palpite == numero_secreto:
        acertou = True
        pontuacao_atual = (MAX_TENTATIVAS - (tentativas_feitas - 1)) * 10 # More points for fewer attempts
        print(f"Parabéns! Você acertou em {tentativas_feitas} tentativas! Sua pontuação é: {pontuacao_atual} pontos.")

        # Update ranking with username, score, and difficulty
        ranking_global.append((nome_jogador, pontuacao_atual, dificuldade_atual))
        # Sort by difficulty (ascending) and then by score (descending)
        ranking_global.sort(key=lambda x: (DIFFICULTY_ORDER.get(x[2], 99), -x[1]))
        ranking_global = ranking_global[:5] # Keep only top 5
        salvar_ranking(ranking_global)

        exibir_ranking(ranking_global, title="Ranking Atualizado")

      elif palpite < numero_secreto:
        print("Tente um número maior.")
        tentativas_restantes -= 1
      else:
        print("Tente um número menor.")
        tentativas_restantes -= 1

      # Add even/odd hint after the first wrong attempt
      if not acertou and tentativas_feitas == 1 and not dica_par_impar_dada:
        if numero_secreto % 2 == 0:
          print("Dica: O número secreto é PAR.")
        else:
          print("Dica: O número secreto é ÍMPAR.")
        dica_par_impar_dada = True

    if tentativas_restantes > 0 and not acertou:
      print(f"Você tem {tentativas_restantes} tentativas restantes.")

  if not acertou:
      print(f"Suas tentativas acabaram! O número secreto era {numero_secreto}.")
      exibir_ranking(ranking_global)

  print("\n--- Fim do Jogo ---")

# Load ranking when the script starts
ranking_global = carregar_ranking()

# Main loop to allow playing multiple times
while True:
  jogar_adivinhacao()
  
  # Option to reset ranking
  while True:
    opcao = input("Deseja jogar novamente (sim/não) ou reiniciar o ranking (reset)? ").lower()
    if opcao == 'sim':
      break
    elif opcao == 'reset':
      resetar_ranking()
      break
    elif opcao == 'não':
      print("Obrigado por jogar! Até a próxima!")
      exit() # Exit the program after saying goodbye
    else:
      print("Opção inválida. Por favor, digite 'sim', 'não' ou 'reset'.")
