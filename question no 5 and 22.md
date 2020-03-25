Q- A university computer science department has a teaching assistant (TA) who helps undergraduate  students with their programming assignments during regular office hours. The TA’s office is rather small and has room for only one desk with a chair and computer. There are three chairs in the hallway outside the office where students can sit and wait if the TA is currently helping another student. When there are no students who need help during office hours, the TA sits at the desk and takes a nap. If a student arrives during office hours and finds the TA sleeping, the student must awaken the TA to ask for help. If a student arrives and finds the TA currently helping another student, the student
sits on one of the chairs in the hallway and waits. If no chairs are available, the student will come back at a later time. Using threads, mutex locks, and semaphores, implement a solution that coordinates the activities of the TA and the students.

 Solution:

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <semaphore.h>

void* student_actions( void* student_id );
void* ta_actions();

#define NUM_WAITING_CHAIRS 3
#define DEFAULT_NUM_STUDENTS 5

sem_t sem_students;
sem_t sem_ta;
pthread_mutex_t mutex_thread;

int waiting_room_chairs[3];
int number_students_waiting = 0;
int next_seating_position = 0;
int next_teaching_position = 0;
int ta_sleeping_flag = 0;

int main( int argc, char **argv ){

	int i;
	int student_num;

	if (argc > 1 ) {
		if ( isNumber( argv[1] ) == 1) {
			student_num = atoi( argv[1] );
		}
		else {
			printf("Invalid input. Quitting program.");
			return 0;
		}
	}
	else {
		student_num = DEFAULT_NUM_STUDENTS;
	}

	int student_ids[student_num];
	pthread_t students[student_num];
	pthread_t ta;

	sem_init( &sem_students, 0, 0 );
	sem_init( &sem_ta, 0, 1 );

	pthread_mutex_init( &mutex_thread, NULL );
	pthread_create( &ta, NULL, ta_actions, NULL );
	for( i = 0; i < student_num; i++ )
	{
		student_ids[i] = i + 1;
		pthread_create( &students[i], NULL, student_actions, (void*) &student_ids[i] );
	}


	pthread_join(ta, NULL);
	for( i =0; i < student_num; i++ )
	{
		pthread_join( students[i],NULL );
	}

	return 0;
}

void* ta_actions() {

	printf( "Checking for students.\n" );

	while( 1 ) {

		if ( number_students_waiting > 0 ) {

			ta_sleeping_flag = 0;
			sem_wait( &sem_students );
			pthread_mutex_lock( &mutex_thread );

			int help_time = rand() % 5;
			printf( "Helping a student for %d seconds. Students waiting = %d.\n", help_time, (number_students_waiting - 1) );
			printf( "Student %d receiving help.\n",waiting_room_chairs[next_teaching_position] );

			waiting_room_chairs[next_teaching_position]=0;
			number_students_waiting--;
			next_teaching_position = ( next_teaching_position + 1 ) % NUM_WAITING_CHAIRS;

			sleep( help_time );

			pthread_mutex_unlock( &mutex_thread );
			sem_post( &sem_ta );

		}
		else {

			if ( ta_sleeping_flag == 0 ) {

				printf( "No students waiting. Sleeping.\n" );
				ta_sleeping_flag = 1;

			}

		}

	}

}

void* student_actions( void* student_id ) {

	int id_student = *(int*)student_id;

	while( 1 ) {
		if ( isWaiting( id_student ) == 1 ) { continue; }
		int time = rand() % 5;
		printf( "\tStudent %d is programming for %d seconds.\n", id_student, time );
		sleep( time );

		pthread_mutex_lock( &mutex_thread );

		if( number_students_waiting < NUM_WAITING_CHAIRS ) {

			waiting_room_chairs[next_seating_position] = id_student;
			number_students_waiting++;
			printf( "\t\tStudent %d takes a seat. Students waiting = %d.\n", id_student, number_students_waiting );
			next_seating_position = ( next_seating_position + 1 ) % NUM_WAITING_CHAIRS;

			pthread_mutex_unlock( &mutex_thread );

			sem_post( &sem_students );
			sem_wait( &sem_ta );

		}
		else {

			pthread_mutex_unlock( &mutex_thread );

		
			printf( "\t\t\tStudent %d will try later.\n",id_student );

		}

	}

}

int isNumber(char number[])
{
    int i;
		for ( i = 0 ; number[i] != 0; i++ )
    {
        if (!isdigit(number[i]))
            return 0;
    }
    return 1;
}

int isWaiting( int student_id ) {
	int i;
	for ( i = 0; i < 3; i++ ) {
		if ( waiting_room_chairs[i] == student_id ) { return 1; }
	}
	return 0;
}

Q-Write a program that implements the FIFO page replacement algorithm. First, generate a random page-reference string where page numbers range from 0 to 9. Apply the random page-reference string to each algorithm, and record the number of page faults incurred by each algorithm. Implement the replacement algorithm so that the number of page frames can vary from 1 to 7. Assume that demand paging is used.
Solution:
#include<stdio.h>
int main()
{
int i,j,n,a[10],frame[8],no,k,avail,count=0;
            printf("\n ENTER THE NUMBER OF PAGES:\n");
scanf("%d",&n);
            printf("\n ENTER THE PAGE NUMBER :\n");
            for(i=1;i<=n;i++)
            scanf("%d",&a[i]);
            printf("\n ENTER THE NUMBER OF FRAMES :");
            scanf("%d",&no);
for(i=0;i<no;i++)
            frame[i]= -1;
                        j=0;
                        printf("\tref string\t page frames\n");
for(i=1;i<=n;i++)
                        {
                                    printf("%d\t\t",a[i]);
                                    avail=0;
                                    for(k=0;k<no;k++)
if(frame[k]==a[i])
                                                avail=1;
                                    if (avail==0)
                                    {
                                                frame[j]=a[i];
                                                j=(j+1)%no;
                                                count++;
                                                for(k=1;k<no;k++)
                                                printf("%d\t",frame[k]);
}
                                    printf("\n");
}
                        printf("Page Fault Is %d",count);
                        return 0;
}
