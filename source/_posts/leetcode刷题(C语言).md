---
title: leetcode刷题(C语言)
date: 2019-01-01 00:00:00
tags:
---

## #9. Palindrome Number
 
> Determine whether an integer is a palindrome. An integer is a
> palindrome when it reads the same backward as forward.

>   Example 1:
>     
>    Input: 121
>    Output: true
>    Example 2:
>     
>    Input: -121
>    Output: false
>    Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
>    Example 3:
>     
>    Input: 10
>    Output: false
>    Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
>    Follow up:
>     
>Coud you solve it without converting the integer to a string?
<!--more-->

```c
bool isPalindrome(int x) {
    if(x<0)
        return false;
    else{
        int nums[999];
        int i=0;
        while(x!=0){
            nums[i]=x%10;
            x=x/10;
            i++;
        }
        for(int j=0;j<i/2;j++){
            if(nums[j]!=nums[i-j-1]){
                return false;
                break;
            }
        }
        return true;
    }
}
```
## #21. Merge Two Sorted Lists

>  Merge two sorted linked lists and return it as a new list. The new
> list should be made by splicing together the nodes of the first two
> lists.
> 
> Example:
> 
> Input: 1->2->4, 1->3->4 Output: 1->1->2->3->4->4
>

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* mergeTwoLists(struct ListNode* l1, struct ListNode* l2) {
    struct ListNode *p,*q,*newlist,*r;
    p=l1;q=l2;
    if(p==NULL){
        return q;
    }else if(q==NULL){
        return p;
    }
    if(l2->val > l1->val)
    {
        newlist=l1;
        p=l1->next;
    }else{
        newlist=l2;
        q=l2->next;
    }
    r=newlist;
    while(p!=NULL && q!=NULL){
        if(p->val > q->val)
        {
            r->next=q;
            q=q->next;
        }else{
            r->next=p;
            p=p->next;
        }
        r=r->next;
    }
    if(p==NULL){
        r->next=q;
    }else{
        r->next=p;
    }
    return newlist;
    
}
```
## #796. Rotate String

> We are given two strings, A and B.
> 
> A shift on A consists of taking string A and moving the leftmost
> character to the rightmost position. For example, if A = 'abcde', then
> it will be 'bcdea' after one shift on A. Return True if and only if A
> can become B after some number of shifts on A.
> 
> Example 1: Input: A = 'abcde', B = 'cdeab' Output: true
> 
> Example 2: Input: A = 'abcde', B = 'abced' Output: false Note:
> 
> A and B will have length at most 100.

```c
bool rotateString(char* A, char* B) {
    int lena=strlen(A);
    int lenb=strlen(B);
    char *newa=A;
    char tempc;
    int j=0;
    if(lenb!=lena)
        return false;
    else if(strcmp(A,B)==0){
        return true;
    }
    //printf("%s\n", A);
    while(j<lena){
        tempc=newa[0];
        for (int i = 0; i < lena-1; i++)
        {
            newa[i]=newa[i+1];
        }
        newa[lena-1]=tempc;
        //printf("%s,%s\n",newa,B );
        if(strcmp(B,newa)==0){
            //printf("find\n");
            return true;
        }
        j++;
    }
    return false;
    
}
```

