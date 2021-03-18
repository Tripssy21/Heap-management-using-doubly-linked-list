#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#define HEAPSIZE 100

//Node for Allocation List
typedef struct NodeAllocate
{
    char name[10];
    int StartingIndex, OccupiedSpace;
    struct NodeAllocate *next, *prev;
} NodeAllocate;

//Declaring pointer variable for Allocation List
NodeAllocate *AllocHead, *AllocTail;

//Node for Free list
typedef struct NodeFree
{
    int StartingIndex, FreeSpace;
    struct NodeFree *next, *prev;
} NodeFree;

//Declaring pointer variable for Free List
NodeFree *FreeHead, *FreeTail;

//Initial condition i.e, before allocation of any memory
void initialize()
{
    AllocHead = NULL;
    AllocTail = NULL;
    NodeFree *Node;
    Node = (NodeFree *)malloc(sizeof(NodeFree));
    Node->StartingIndex = 0;
    Node->FreeSpace = HEAPSIZE;
    Node->next = NULL;
    Node->prev = NULL;
    FreeHead = Node;
    FreeTail = Node;
}

//Function to print out free list and allocated list
void PrintList(void)
{
    NodeAllocate *tempAlloc = AllocHead;
    NodeFree *tempFree = FreeHead;
    printf("The Free List is :\n");
    while (tempFree)
    {
        printf("\tStarting Address :%d\tEnding Address:%d\n", tempFree->StartingIndex, (tempFree->FreeSpace + tempFree->StartingIndex - 1));
        tempFree = tempFree->next;
    }
    printf("\n");
    printf("The Allocated List is :\n");
    while (tempAlloc)
    {
        printf("\tProcess Name : %s\tStarting Address :%d\tEnding Address:%d\n", tempAlloc->name, (tempAlloc->StartingIndex, tempAlloc->OccupiedSpace + tempFree->StartingIndex - 1));
        tempAlloc = tempAlloc->next;
    }
}

//Allocation of heap memory
void ManuallyAllocate(int size, char *str)
{
    int flag = 1;
    NodeFree *temp = FreeHead;
    while (flag && temp->next)
    {
        if (temp->FreeSpace > size)
        {
            temp->FreeSpace -= size;
            NodeAllocate *NewAlloc = (NodeAllocate *)malloc(sizeof(NodeAllocate));
            NewAlloc->next = NULL;
            NewAlloc->OccupiedSpace = size;
            NewAlloc->StartingIndex = temp->StartingIndex;
            strcpy(NewAlloc->name, str);
            flag = 0;
            if (!AllocHead)
            {
                AllocHead = NewAlloc;
                AllocTail = NewAlloc;
                NewAlloc->prev = NULL;
            }
            else
            {
                NewAlloc->prev = AllocTail;
                AllocTail = NewAlloc;
            }
        }
        else
        {
            temp = temp->next;
        }
    }
    if (flag)
    {
        printf("Not enough Heap meamory is Free");
    }
}

//Freeing allocated memory
void FreeMember(char *str)
{
    NodeAllocate *temp1 = AllocHead;
    while (strcmp(str, temp1->name) && temp1 != AllocTail)
    {
        temp1 = temp1->next;
    }
    if (temp1->next)
    {
        if (temp1->prev)
        {
            AllocHead = temp1->next;
            AllocHead->prev = NULL;
        }
        else
        {
            temp1->prev->next = temp1->next;
            temp1->next->prev = temp1->prev;
        }
        NodeFree *temp2 = FreeHead;
        while (temp2->StartingIndex < (temp1->StartingIndex + temp1->OccupiedSpace - 1) && temp2->next)
        {
            temp2 = temp2->next;
        }
        if (temp2->StartingIndex == (temp1->StartingIndex + temp1->OccupiedSpace))
        {
            temp2->StartingIndex -= temp1->OccupiedSpace;
            temp2->FreeSpace += temp1->OccupiedSpace;
        }
        else if (temp1->StartingIndex == (temp2->prev->StartingIndex + temp2->prev->FreeSpace))
        {
            temp2->FreeSpace += temp1->OccupiedSpace;
        }
        else
        {
            NodeFree *temp3 = (NodeFree *)malloc(sizeof(NodeFree));
            temp3->next = temp2;
            temp3->prev = temp2->prev;
            temp3->FreeSpace = temp1->OccupiedSpace;
            temp3->StartingIndex = temp1->StartingIndex;
            temp2->prev->next = temp3;
            temp2->prev = temp3;
        }
        free(temp1);
    }
}

int main()
{
    initialize();
    int size = 0, choice = 1;
    char name[10];
    while (choice)
    {
        printf("\nGive choice\n");
        printf("1.Allocation\n2.Free Memory\n3.Print Lists\n4.Exit\n");
        printf("Choice : ");
        scanf("%d", &choice);
        switch (choice)
        {
        case 1:
            printf("Enter the process name : ");
            scanf("%s", name);
            printf("Enter the no. of the block to allocate : ");
            scanf("%d", &size);
            ManuallyAllocate(size, name);
            printf("\n");
            break;

        case 2:
            printf("Enter the process to be freed : ");
            scanf("%s", name);
            FreeMember(name);
            printf("\n");
            break;

        case 3:
            PrintList();
            break;

        case 4:
            exit(0);
            break;

        default:
            printf("Invaid Choice, please try again\n");
        }
    }
    return 0;
}
