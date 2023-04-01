####################################################################################################
# Global imports
####################################################################################################
from datetime import datetime
from twilio.rest import Client
import pandas as pd
import random
import csv
import time


####################################################################################################
# Base functions
####################################################################################################
# Function providing access to Twilio's API
def twilio_account_verification():
    account_sid = "[INSERT TWILIO ACCOUNT ID HERE]"
    auth_token = "[INSERT TWILIO TOKEN HERE]"
    client = Client(account_sid, auth_token)
    return client


# Function using Twilio's API to send a text message to a phone number
def send_message(twilio_client, out_number, message):
    out_number = "+" + str(out_number)
    message_to_send = twilio_client.messages.create(
        to=out_number,
        from_="+[INSERT TWILIO PHONE NUMBER HERE]",
        body=message)
    print(message_to_send.sid)


# Function that returns the current date and time
def get_date_and_time():
    date_time = str(datetime.now())
    current_date, current_time = date_time.split()[0], date_time.split()[1]
    return current_date, current_time


# Function that returns a list of intervention phone numbers
def get_phone_book():
    phone_book = pd.read_csv('phone_book.csv')
    return phone_book


# Function that returns phone numbers matching the current time
def preferred_base_intervention_message_time():
    phone_book = get_phone_book()
    current_date, current_time = get_date_and_time()
    current_time = current_time[0:4] # extracts the hour and tens digit of the current time
    current_hour_phone_book = phone_book.loc[phone_book.pref_base_intervention_time.str.contains(current_time), :]
    return current_hour_phone_book


# Function that prints to console what messages are being sent to participants
def print_message_to_screen(number, message_to_send, current_time,
                            current_date, number_of_messages_received, message_type):

    print('-' * 80)
    print('Sending', number, "the following message:")
    print(message_to_send)
    print('Message sent at:', current_time, "on", current_date)
    print('This was a', message_type, 'type of message')
    print('This was message number', number_of_messages_received, 'this phone number has received')
    print('-' * 80)
    print('\n')


####################################################################################################
# Introduction message functions
####################################################################################################
# Function that checks if the intervention welcome message was already sent
def introduction_message_already_sent(number):
    history_of_introduction_messages = pd.read_csv('history_of_introduction_messages.csv')
    if number in list(history_of_introduction_messages.phone_numbers):
        return True
    else:
        return False


# Function that checks if the intervention welcome message was already sent today
def introduction_message_already_sent_today(number, current_date):
    history_of_introduction_messages = pd.read_csv('history_of_introduction_messages.csv')
    if len(history_of_introduction_messages.loc[(history_of_introduction_messages.phone_numbers == number)
                                                & (history_of_introduction_messages.date_sent.str.contains(str(current_date))), :]) > 0:
        print('already sent today, sorry!')
        return True
    else:
        print('did not send today')
        return False


# Function that writes sent message to the history of messages sent .csv file
def write_sent_intro_message_data(message_to_send, number, current_date, current_time, message_type):
    row = [message_to_send, number, current_date, current_time, message_type]

    with open('history_of_introduction_messages.csv', 'a') as f:
        writer = csv.writer(f)
        writer.writerow(row)


####################################################################################################
# Conclusion message functions
####################################################################################################
# Function that checks if the intervention conclusion message was already sent
def conclusion_message_already_sent(number):
    history_of_conclusion_messages = pd.read_csv('history_of_conclusion_messages.csv')
    if number in list(history_of_conclusion_messages.phone_numbers):
        return True
    else:
        return False


# Function that checks if the intervention concluded today
def intervention_concluded_today(number_df, current_date):
    if len(number_df.loc[number_df.date_sent == current_date, :]) > 0:
        return True
    else:
        return False


# Function that writes sent message to the history of conclusions messages .csv file
def write_sent_conclusion_message_data(message_to_send, number, current_date, current_time, message_type):
    row = [message_to_send, number, current_date, current_time, message_type]

    with open('history_of_conclusion_messages.csv', 'a') as f:
        writer = csv.writer(f)
        writer.writerow(row)


####################################################################################################
# Primary intervention message functions
####################################################################################################
# Function that pulls a list of intervention message sets to send participants
def pool_of_intervention_messages():
    file_names = ['set_1.csv', 'set_2.csv', 'set_3.csv']
    sets = []

    for file in file_names:
        set = pd.read_csv(file)
        sets.append(set)

    set_1 = sets[0]
    set_2 = sets[1]
    set_3 = sets[2]

    return set_1, set_2, set_3


# Function that opens a file of all previously sent base intervention messages
def history_of_base_intervention_messages():
    base_intervention_message_history = pd.read_csv('history_of_base_intervention_messages.csv')
    return base_intervention_message_history


# Function that subsets a new data frame for a given phone number
def get_data_frame_of_base_intervention_phone_number(number):
    base_intervention_message_history = history_of_base_intervention_messages()
    number_df = base_intervention_message_history.loc[base_intervention_message_history.phone_numbers == number]
    number_of_messages_received = []

    for row in number_df.number_of_messages_received:
        number_of_messages_received.append(row)

    if len(number_of_messages_received) == 0:
        number_of_messages_received = [0]

    return number_df, number_of_messages_received[-1]


# Function that determines if a number has already received base intervention messages in the day
def base_intervention_message_sent_today(number_df, current_date):
    if len(number_df.loc[number_df.date_sent.str.contains(current_date), :]) > 0:
        return True
    else:
        return False


# Function that determines which message set to pull from
def determine_message_set(number_df, set_1, set_2, set_3):
    if sum(number_df.comp_message_set_1) <= 9:
        cont = True
        message_set = set_1
        set_values = [1, 0, 0]

    elif sum(number_df.comp_message_set_2) <= 9:
        cont = True
        message_set = set_2
        set_values = [0, 1, 0]

    elif sum(number_df.comp_message_set_3) <= 7:
        cont = True
        message_set = set_3
        set_values = [0, 0, 1]

    else:
        cont = False
        message_set = False
        set_values = False

    return cont, message_set, set_values


# Function that selects a random message to send a participant (without repeat)
def get_message_to_send(number_df, message_set):
    random_message = random.choice(list(message_set.message_1))

    while random_message in list(number_df.message_sent):
        random_message = random.choice(list(message_set.message_1))

    current_message_set = message_set.loc[message_set.message_1.str.contains(random_message),:]
    message_1 = list(current_message_set.message_1)[0]
    message_2 = list(current_message_set.message_2)[0]

    return message_1, message_2


# Function that writes sent base intervention messages to the history of base intervention messages sent .csv file
def write_base_intervention_sent_message_data(message_to_send, number, current_date, current_time, number_of_messages_received,
                            message_set_1_value, message_set_2_value, message_set_3_value, message_type):

    row = [message_to_send, number, current_date, current_time, number_of_messages_received,
           message_set_1_value, message_set_2_value, message_set_3_value, message_type]

    with open('history_of_base_intervention_messages.csv', 'a') as f:
        writer = csv.writer(f)
        writer.writerow(row)


####################################################################################################
# Medication reminder message functions
####################################################################################################
# Function that returns phone numbers matching the current time for med reminder participants
def preferred_med_reminder_time():
    phone_book = get_phone_book()
    current_date, current_time = get_date_and_time()
    current_time = current_time[0:4]
    current_hour_phone_book = phone_book.loc[phone_book.pref_med_reminder_time.str.contains(current_time), :]
    return current_hour_phone_book


# Function that opens a file of all previously sent med reminder messages
def history_of_med_reminder_messages():
    message_history = pd.read_csv('history_of_med_reminder_messages.csv')
    return message_history


# Function that subsets a new data frame of a phone number
def get_data_frame_of_med_reminder_phone_number(number):
    message_history = history_of_med_reminder_messages()
    number_df = message_history.loc[message_history.phone_numbers == number]
    number_of_messages_received = []

    for row in number_df.number_of_messages_received:
        number_of_messages_received.append(row)

    if len(number_of_messages_received) == 0:
        number_of_messages_received = [0]

    return number_df, number_of_messages_received[-1]


# Function that determines if a number has already received a med reminder today
def med_reminder_message_sent_today(number_df, current_date):
    if len(number_df.loc[number_df.date_sent.str.contains(current_date), :]) > 0:
        return True
    else:
        return False


# Function that writes sent med reminder message to the history of med reminder messages sent .csv file
def write_med_reminder_sent_message_data(message_to_send, number, current_date, current_time, number_of_messages_received, message_type):
    row = [message_to_send, number, current_date, current_time, number_of_messages_received, message_type]

    with open('history_of_med_reminder_messages.csv', 'a') as f:
        writer = csv.writer(f)
        writer.writerow(row)


####################################################################################################
# Main program
####################################################################################################
def main():
    twilio_client = twilio_account_verification()
    set_1, set_2, set_3 = pool_of_intervention_messages()
    current_date, current_time = get_date_and_time()
    current_base_intervention_time_phone_book = preferred_base_intervention_message_time()

    # Introduction/base intervention code
    for number in current_base_intervention_time_phone_book.phone_numbers:
        try:
            # The program starts by attempting to send a welcome message to participants
            if not introduction_message_already_sent(number):
                phone_book = get_phone_book()
                number_df = phone_book.loc[phone_book.phone_numbers == number, :]
                name = list(number_df.name)[0]

                message_1 = "Hi, " + name + '. Welcome to the Text4IBD program! For the next 2 weeks starting ' \
                                               'tomorrow, you will receive daily text messages from us. The purpose ' \
                                               'of these messages is to provide support about managing IBD.'

                message_2 = 'Please note that this phone number is not actively monitored. If you have a medical ' \
                            'question about IBD, please contact your doctor. '

                messages = [message_1, message_2]
                message_counter = 0

                for message in messages:
                    message_counter += 1
                    message_type = '"Welcome message"'
                    write_sent_intro_message_data(message, number, current_date, current_time, message_type)
                    print_message_to_screen(number, message, current_time, current_date, message_counter, message_type)
                    send_message(twilio_client, number, message)
                    time.sleep(5)

            # If a welcome message was already sent, then it moves on to the base intervention.
            else:
                number_df, number_of_messages_received = get_data_frame_of_base_intervention_phone_number(number)

                if not introduction_message_already_sent_today(number, current_date):

                    if not base_intervention_message_sent_today(number_df, current_date) and int(number_of_messages_received) < 14:
                    # if True and int(number_of_messages_received) < 14:
                        print(number_of_messages_received)
                        cont, message_set, set_values = determine_message_set(number_df, set_1, set_2, set_3)

                        if cont:
                            message_1, message_2 = get_message_to_send(number_df, message_set)
                            messages = [message_1, message_2]

                            for message in messages:

                                if len(number_df) == 0:
                                    number_of_messages_received = 1
                                else:
                                    number_of_messages_received = list(number_df.number_of_messages_received)[-1] + 1

                                message_type = '"Base intervention message"'
                                write_base_intervention_sent_message_data(message, number, current_date, current_time,
                                                                          number_of_messages_received, set_values[0],
                                                                          set_values[1], set_values[2], message_type)

                                print_message_to_screen(number, message, current_time, current_date,
                                                        number_of_messages_received, message_type)

                                send_message(twilio_client, number, message)
                                time.sleep(5)
                        else:
                            continue

                    else:
                        continue

                else:
                    continue

        except Exception as e:
            print('Exception noted:', e)
            continue

    # Med reminder code
    current_med_reminder_time_phone_book = preferred_med_reminder_time()
    for number in current_med_reminder_time_phone_book.phone_numbers:
        try:
            if introduction_message_already_sent(number):
                number_df, number_of_messages_received = get_data_frame_of_med_reminder_phone_number(number)

                if not introduction_message_already_sent_today(number, current_date):

                    if not med_reminder_message_sent_today(number_df, current_date) and int(number_of_messages_received) < 14:
                        # if True and int(number_of_messages_received) < 14:
                        message = "REMINDER: Be sure to take your IBD medication today."

                        if len(number_df) == 0:
                            number_of_messages_received = 1
                        else:
                            number_of_messages_received = list(number_df.number_of_messages_received)[-1] + 1

                        message_type = '"Med reminder message"'
                        write_med_reminder_sent_message_data(message, number, current_date, current_time, number_of_messages_received, message_type)
                        print_message_to_screen(number, message, current_time, current_date, number_of_messages_received, message_type)
                        send_message(twilio_client, number, message)

                    else:
                        continue

                else:
                    continue

            else:
                continue

        except Exception as e:
            print('Exception noted:', e)
            continue

    # Intervention conclusion code
    for number in current_base_intervention_time_phone_book.phone_numbers:
        try:
            number_df, number_of_messages_received = get_data_frame_of_base_intervention_phone_number(number)

            if int(number_of_messages_received) == 14:

                if not conclusion_message_already_sent(number):

                    if not intervention_concluded_today(number_df, current_date):
                        conclusion_message = 'Thank you for participating in Text4IBD. You have finished the text message ' \
                                             'portion of the program. Please keep an eye out in your email for instructions ' \
                                             'about how to take the final survey.'

                        write_sent_conclusion_message_data(conclusion_message, number, current_date, current_time,
                                                           'Conclusion_message')

                        print_message_to_screen(number, conclusion_message, current_time, current_date, 1,
                                                'conclusion message')

                        send_message(twilio_client, number, conclusion_message)

            else:
                continue

        except Exception as e:
            print('Exception noted:', e)
            continue


main()

