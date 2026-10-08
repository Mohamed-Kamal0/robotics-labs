#!/usr/bin/env python3
import rospy
from geometry_msgs.msg import Twist

def move_turtle():
    # Initialize the ROS node
    rospy.init_node('move_turtle', anonymous=True)
    
    # Create publisher for /turtle1/cmd_vel topic
    pub = rospy.Publisher('/turtle1/cmd_vel', Twist, queue_size=10)
    
    rate = rospy.Rate(10)  # 10 Hz loop rate
    
    start_time = rospy.get_time()
    
    # Toggle state: True = X-axis (horizontal), False = Y-axis (vertical)
    move_x = True
    
    rospy.loginfo("Turtle controller node started. Toggling motion every 2.5 seconds.")
    
    while not rospy.is_shutdown():
        current_time = rospy.get_time()
        
        # Switch axis every 2.5 seconds
        if current_time - start_time >= 2.5:
            move_x = not move_x
            start_time = current_time
            axis_name = "X (horizontal)" if move_x else "Y (vertical)"
            rospy.loginfo("Switched movement to %s axis", axis_name)
            
        vel_msg = Twist()
        
        if move_x:
            vel_msg.linear.x = 0.5
            vel_msg.linear.y = 0.0
        else:
            vel_msg.linear.x = 0.0
            vel_msg.linear.y = 0.5
            
        pub.publish(vel_msg)
        rate.sleep()

if __name__ == '__main__':
    try:
        move_turtle()
    except rospy.ROSInterruptException:
        pass
