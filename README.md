# Fold-em
# made by Pawel Tempczyk 2018 ptempczyk@gmail.com

import random
import collections
from enum import Enum
import os
import pandas as pd
import math
from scipy.stats.stats import pearsonr

def starting():
    print()
    text = """You assess your chance of winning taking into consideration two factors: cards and behavior of your opponents.
    FoldEm improves your poker skills by verifying your winning-loosing projections only basing on cards.
    If you are not familiar with Texas Hold'em rules type "r" and press Enter.
    If you would like to scan the hierarchy of combinations type "h" and press Enter.
    Each round cards are randomly selected from the deck. While playing PokeHer you make a guess the number of opponents you will win with. Each round you make four guesses as you see more cards. At the end you will be asked how certain you are of your guesses.
    Remember that tying doesn't count as winning. If you and your opponent's rankings differ the decision who won is based on the hierarchy of combinations. If they are the same you will see the opponent's highest five cards and the decision who won.
    At the end you will see the table presenting the accuracy of your guesses. You will have the chance to observe how your accuracy rises (or falls) if more cards are visible.
    Your accuracy is max (100%) when you guess the exact number of wins and min (0%) when you make the most extreme mistake. So in game with 5 opponents and 3 wins if you guess 3, you have max, and if 0, you get min accuracy. """
    print(text)
    button = input('Press Enter to go further')
    return button

def presenting_rules():
    print()
    text = """Texas Hold'em is the most popular poker variation. Your aim is to find the highest five cards among seven (two from your hand and five from the table).
    There are four phases of making bets:
    1. After seeing your hand (two cards)
    2. After "flop" (three cards on the table)
    3. After "turn" (the fourth card delt)
    4. After "river" (the fifth card delt)
    The highest five is always combination (like pair, two pairs etc) and other highest cards complementing to five. To determine who wins you compare each card starting with combinations.
    With hand: Q 4 and table: Q J 3 9 3 6 you would have combination: two pairs (of Qs and of 3s) and highest hand: Q Q 3 3 9.
    With hand: 5 8 and table: 9 A 10 7 6 you have straight and highest hand: 6 7 8 9 10.
    If five highest cards of two gamers are the same you tie. But remeber it doesn't happen frequently :)
    More information you can find here: https://www.pokernews.com/poker-rules/texas-holdem.htm """
    print(text)
    button = input('Press Enter to go back')
    return button

def presenting_combinations():
    print()
    text = """Combinations are shown from the highest ranking to the lowest.
    1. Royal Flush — five cards of the same suit, ranked ace through ten
    2. Straight Flush — five cards of the same suit and consecutively ranked
    3. Four of a Kind — four cards of the same rank
    4. Full House — three cards of the same rank and two more cards of the same rank
    5. Flush — any five cards of the same suit
    6. Straight — any five cards consecutively ranked
    7. Three of a Kind — three cards of the same rank
    8. Two Pair — two cards of the same rank and two more cards of the same rank
    9. One Pair — two cards of the same rank
    10. High Card — five unmatched cards
    Stolen from: https://www.pokernews.com/poker-rules/texas-holdem.htm"""
    print(text)
    button = input('Press Enter go back')
    return button

def number_of_opponents():
    print()
    print('Now write the number of opponents you will play with ')
    button = input('Press a number from 1 to 9, preferably 5 or 6, and press Enter ')
    try: button = int(button)
    except:
        ValueError
        print(r'My dear, it wasn\'t a number! Try again')
        number_of_opponents()
    if button < 0 or button > 9:
        print('Number of opponents has to be higher than 0 and lower than 10! Try again')
        number_of_opponents()
    return button

def number_of_rounds():
    print()
    print('Now write the number of rounds you will play ')
    button = input('Press a number from 1 to 20, preferably 5, and press Enter ')
    try: button = int(button)
    except:
        ValueError
        print(r'My dear, it wasn\'t a number! Try again')
        number_of_rounds()
    if button < 0 or button > 20:
        print('Number of rounds has to be higher than 0 and lower than 21! Try again')
        number_of_rounds()
    return button

def introduction():
    button = starting()
    if button == 'r':
        button = presenting_rules()
        button = starting()
    elif button == 'h':
        button = presenting_combinations()
        button = starting()
    else:
        pass

def predicted_number_of_wins_table(no_of_rounds):
    predicted_no_of_wins = dict()
    for phases in range(4):
        predicted_no_of_wins[phases] = [None] * no_of_rounds
    return predicted_no_of_wins

def accuracy_table(no_of_rounds):
    accuracy = dict()
    for phases in range(4):
        accuracy[phases] = [None] * no_of_rounds
    return accuracy

def certainty_table(no_of_rounds):
    certainty = [None] * no_of_rounds
    return certainty

def number_of_wins_per_round(opponents):
    sum = 0
    for opponent in opponents:
        if who_wins('me', opponent) == 1:
            sum += 1
    return sum

def number_of_wins_table(no_of_rounds):
    no_of_wins = [None] * no_of_rounds
    return no_of_wins

def predicting_number_of_wins():
    predicted_no_of_wins = input('Now guess a number of opponents you will win with and press Enter ')
    try: int(predicted_no_of_wins)
    except:
        ValueError
        print(r'My dear, it wasn\'t a number! Try again')
        predicting_number_of_wins()
    if int(predicted_no_of_wins) < 0 or int(predicted_no_of_wins) > no_of_opponents:
        print('This number should be at least 0 and at most', no_of_opponents,'! Try again.')
        predicting_number_of_wins()
    return int(predicted_no_of_wins)

def telling_certainty():
    certainty = input('Now tell how certain are you with your prediction (percent) ')
    try: return int(certainty)
    except:
        ValueError
        print(r'My dear, it wasn\'t a number! Try again')
        telling_certainty()

class Card:
    def __init__(self, rank, suit):
        self.rank = rank
        self.suit = suit

    def __repr__(self):
        return "%s%s" % (self.rank, chr(self.suit))

    def getRank(self):
        return self.rank

    def getSuit(self):
        return self.suit


rank_table = ["2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"]
suit_table = [9831, 9826, 9825, 9828]

full_deck = [None] * 52
i = 0
for rank in rank_table:
    for suit in suit_table:
        full_deck[i] = Card(rank, suit)
        i += 1

def handing_out_cards(full_deck, players):
    deck = full_deck.copy()
    cards_from = dict()

    cards_from['table'] = []
    for player in players:
        cards_from[player] = []

    for player in players:
        for i in range(2):
            card = random.choice(deck)
            cards_from[player].append(card)
            deck.remove(card)

    # five cards for the table are already randomly selected and waiting to be shown all
    for i in range(5):
        card = random.choice(deck)
        cards_from['table'].append(card)
        deck.remove(card)
    return cards_from

class Combinations(Enum):
    ROYAL_FLUSH = 1
    STRAIGHT_FLUSH = 2
    FOUR_OF_A_KIND = 3
    FULL_HOUSE = 4
    FLUSH = 5
    STRAIGHT = 6
    THREE_OF_A_KIND = 7
    TWO_PAIR = 8
    ONE_PAIR = 9
    HIGH_CARD = 10

def preparing_ranks(cards_from, players):
    ranks = dict()
    index_ranks = dict()

    for player in players:
        ranks[player] = list()
        index_ranks[player] = list()

        for card in cards_from['table'] + cards_from[player]:
            ranks[player].append(card.rank)
        for element in ranks[player]:
            index_ranks[player].append(rank_table.index(element))
        index_ranks[player] = list(reversed(sorted(index_ranks[player])))
    return index_ranks

def preparing_suits(cards_from, players):
    suits = dict()

    for player in players:
        suits[player] = list()

        for card in cards_from['table'] + cards_from[player]:
            suits[player].append(card.suit)
        suits[player] = list(reversed(sorted(suits[player])))
    return suits

def ranking(player):
    index_ranks = preparing_ranks(cards_from, players)
    suits = preparing_suits(cards_from, players)

    counter_ranks = dict()
    counter_ranks[player] = list()
    counter_ranks[player] = collections.Counter(index_ranks[player])

    counter_suits = dict()
    counter_suits[player] = list()
    counter_suits[player] = collections.Counter(suits[player])

    def is_color(player):
        if max(counter_suits[player].values()) >= 5:
            return True
        else:
            return False

    def is_straight(player):
        ascending_by_one = 0
        global highest_card_from_street
        highest_card_from_street = 0
        for x in range(len(unique_index_ranks[player]) - 1):
            if unique_index_ranks[player][x + 1] - unique_index_ranks[player][x] == 1:
                highest_card_from_street = unique_index_ranks[player][x + 1]
                ascending_by_one += 1
                if ascending_by_one >= 4:
                    return True
            else:
                ascending_by_one = 0
        return False

    # basing on ranking of combinations in poker and its probability (starting from the most common)
    if max(counter_ranks[player].values()) == 4:
        ranking = Combinations.FOUR_OF_A_KIND
    elif (is_straight(player) == False) and (is_color(player) == False):
        if len(unique_index_ranks[player]) == 7:
            ranking = Combinations.HIGH_CARD
        elif len(unique_index_ranks[player]) == 6:
            ranking = Combinations.ONE_PAIR
        elif max(counter_ranks[player].values()) == 2:
            ranking = Combinations.TWO_PAIR
        elif max(counter_ranks[player].values()) == 3:
            if 2 not in counter_ranks[player].values():
                ranking = Combinations.THREE_OF_A_KIND
            else:
                ranking = Combinations.FULL_HOUSE    
    elif is_straight(player) == True:
        if is_color(player) == False:
            ranking = Combinations.STRAIGHT
        else:
            if highest_card_from_street == 12:
                ranking = Combinations.ROYAL_FLUSH
            else:
                ranking = Combinations.STRAIGHT_FLUSH
    else:
        ranking = Combinations.FLUSH

    return ranking

def winning(number1, number2):
        if number1 > number2:
            return 1
        elif number1 < number2:
            return -1
        else:
            None

def preparing_unique_index_ranks(index_ranks):
    unique_index_ranks = dict()
    for player in players:
        unique_index_ranks[player] = []
        unique_index_ranks[player] = list(set(index_ranks[player]))
    return unique_index_ranks

def preparing_repeating_indexes(index_ranks, unique_index_ranks):
    repeating_indexes = dict()
    for player in players:
        repeating_indexes[player] = list()
        repeating_indexes[player] = index_ranks[player].copy()
        for element in unique_index_ranks[player]:
            repeating_indexes[player].remove(element)
    return repeating_indexes

def combinations(ranking_player1, ranking_player2, player1, player2, repeating_indexes, index_ranks, unique_index_ranks):
    if ranking_player1 == ranking_player2:
        ranking = ranking_player1
        combination = dict()
        try:
            for player in players:
                combination[player] = dict()
                for number in [1,2]:
                    combination[player][number] = list()

            #if ranking('me') == Combinations.HIGH_CARD:
            #    pass

            if ranking == Combinations.ONE_PAIR:
                for player in [player1, player2]:
                    combination[player][1] = [repeating_indexes[player][0]] * 2

            if ranking == Combinations.TWO_PAIR:
                for player in [player1, player2]:
                    combination[player][1] = [repeating_indexes[player][0]] * 2
                    combination[player][2] = [repeating_indexes[player][1]] * 2

            if ranking == Combinations.THREE_OF_A_KIND:
                for player in [player1, player2]:
                    combination[player][1] = [repeating_indexes[player][0]] * 3

            if ranking == Combinations.STRAIGHT:
                ascending_by_one = 0
                for player in [player1, player2]:
                    for x in range(len(unique_index_ranks[player])):
                        if unique_index_ranks[player][x] - unique_index_ranks[player][x + 1] == 1:
                            if ascending_by_one == 0:
                                combination[1][player] = [unique_index_ranks[player][x], unique_index_ranks[player][x+1],
                                unique_index_ranks[player][x+2], unique_index_ranks[player][x+3], unique_index_ranks[player][x+4]]
                            ascending_by_one += 1
                        else:
                            if ascending_by_one < 4:
                                ascending_by_one = 0

            if ranking == Combinations.FULL_HOUSE:
                highest[player] = int
                for player in [player1, player2]:
                    highest[player] = repeating_indexes[player].pop(0)
                if highest[player] == repeating_indexes[player][-1]:
                    combination[player][1] = repeating_indexes[player][-1]
                    combination[player][2] = repeating_indexes[player][-2]
                else:
                    combination[player][2] = repeating_indexes[player][-1]
                    combination[player][1] = repeating_indexes[player][-2]

            if ranking == Combinations.FOUR_OF_A_KIND:
                for player in [player1, player2]:
                    combination[player][1] = repeating_indexes[player][-3]
        except:IndexError
        return combination

def who_wins(player1, player2):
    #checking whether it's possible to determine who is the winner basing on difference of rankings
    global ranking_player1
    global ranking_player2
    ranking_player1 = ranking(player1)
    ranking_player2 = ranking(player2)

    if winning(-ranking_player1.value, -ranking_player2.value) != None:
        return winning(-ranking_player1.value, -ranking_player2.value)

    #if not, checking the highest five cards
    index_ranks = preparing_ranks(cards_from, players)
    highest_five = dict()
    combination = combinations(ranking_player1, ranking_player2, player1, player2, repeating_indexes, index_ranks, unique_index_ranks)

    for player in [player1, player2]:
        highest_five[player] = list()
        for i in [1,2]:
            for x in range(len(combination[player][i])):
                index_ranks[player].remove(combination[player][i][x])
        highest_five[player].extend(combination[player][1])
        highest_five[player].extend(combination[player][2])
        highest_five[player].extend(index_ranks[player][0:(5-len(combination[player][1])-len(combination[player][2]))])

    print ('You have the same ranking with', player2, ', so the highest five cards are from your opponent are', rank_table[highest_five[player2][0]], rank_table[highest_five[player2][1]],
    rank_table[highest_five[player2][2]], rank_table[highest_five[player2][3]], rank_table[highest_five[player2][4]])

    for x in range(5):
        if winning(highest_five[player1][x], highest_five[player2][x]) != None:
            if winning(highest_five[player1][x], highest_five[player2][x]) == 1:
                print('you won')
            if winning(highest_five[player1][x], highest_five[player2][x]) == -1:
                print('opponent won')
            return winning(highest_five[player1][x], highest_five[player2][x])
    print('you tie!')
    return 0

def showing_cards(phases, cards_from, opponents, ranking, showdown=False):
    print('Your cards: ')
    print(cards_from['me'][0], " ", cards_from['me'][1])

    if phases != 0:
        print('Cards from table :')
        for x in range (0, 2 + phases):
            print(cards_from['table'][x])
    if showdown == True:
        print('Time for showdown!')
        print(r'Opponents\' cards and rankings: ')
        for opponent in opponents:
            print(opponent, cards_from[opponent][0], " ", cards_from[opponent][1], "   ", ranking(opponent))

def table(no_of_rounds, predicted_no_of_wins, no_of_wins, certainty):
    for rounds in range(no_of_rounds):
        for phases in range(4):
            accuracy[phases][rounds] = int(100 * (1 - abs(predicted_no_of_wins[phases][rounds] - no_of_wins[rounds]) / max(no_of_wins[rounds], no_of_opponents - no_of_wins[rounds])))
    rounds = list(range(1, no_of_rounds + 1))
    
    rounds.insert(0, "rounds:")
    accuracy[0].insert(0, "hand")
    accuracy[1].insert(0, "flop")
    accuracy[2].insert(0, "turn")
    accuracy[3].insert(0, "river")
    certainty.insert(0, "certainty")

    table = [None] * 6
    table[0] = rounds
    table[1] = accuracy[0]
    table[2] = accuracy[1]
    table[3] = accuracy[2]
    table[4] = accuracy[3]
    table[5] = certainty
    
    results = pd.DataFrame(table)

    print(results)

def summary(predicted_no_of_wins, no_of_wins):

    print()
    print('Good games! Now time for some numbers. Here is the table showing your accuracy (in percent) in 4 phases (hand, flop, turn and river) and your certainty')
    table(no_of_rounds, predicted_no_of_wins, no_of_wins, certainty)
    print('\nAnd now correlations between your guesses and number of wins')
    print('Hand:', '%.2f' % pearsonr(predicted_no_of_wins[0], no_of_wins)[0])
    print('Flop:', '%.2f' % pearsonr(predicted_no_of_wins[1], no_of_wins)[0])
    print('Turn:', '%.2f' % pearsonr(predicted_no_of_wins[2], no_of_wins)[0])
    print('River:', '%.2f' % pearsonr(predicted_no_of_wins[3], no_of_wins)[0])
    print('Generally the correlation should rise if you see more cards. If not you can check in which phase your guesses were closest to achieved wins.')
    print('Thank you for your game! :) made by Pawel Tempczyk ptempczyk@gmail.com')
    
# main
introduction()
no_of_opponents = number_of_opponents()
opponents = ['AI1', 'AI2', 'AI3', 'AI4', 'AI5', 'AI6', 'AI7', 'AI8', 'AI9', 'AI10']
opponents = opponents[0:no_of_opponents]
players = opponents.copy()
players.append('me')
no_of_rounds = number_of_rounds()
print('So you will play', no_of_rounds, 'rounds with', no_of_opponents,'opponents.')
predicted_no_of_wins = predicted_number_of_wins_table(no_of_rounds)
no_of_wins = number_of_wins_table(no_of_rounds)
certainty = certainty_table(no_of_rounds)
accuracy = accuracy_table(no_of_rounds)

for rounds in range(no_of_rounds):
    for phases in range(4):
        print()
        if phases == 0:
            print(rounds + 1,". round out of", no_of_rounds, '.')
            cards_from = handing_out_cards(full_deck, players)
            index_ranks = preparing_ranks(cards_from, players)
            unique_index_ranks = preparing_unique_index_ranks(index_ranks)
            repeating_indexes = preparing_repeating_indexes(index_ranks, unique_index_ranks)

        showing_cards(phases, cards_from, opponents, ranking)
        predicted_no_of_wins[phases][rounds] = predicting_number_of_wins()
        if phases == 3:
            certainty[rounds] = telling_certainty()
            showing_cards(3, cards_from, opponents, ranking,showdown = True)
            print('Your combination is ', ranking('me'),'.')
            no_of_wins[rounds] = number_of_wins_per_round(opponents)
            print('You had ', no_of_wins[rounds], ' wins.')
            forward = input('Press Enter to go further ')

summary(predicted_no_of_wins, no_of_wins)
